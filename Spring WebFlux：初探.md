## Spring WebFlux：初探

原文链接：https://dzone.com/articles/spring-webflux-first-steps

作者：[Biju Kunjummen](https://dzone.com/users/1229843/biju.kunjummen.html)

译者：[jackzhaiyh](https://github.com/jackzhaiyh)

Spring 5 新增WebFlux——在Spring应用中支持响应式编程实践。下面看一下如何将其引入旧版注解风格Controller中。

Spring WebFlux用于表示Spring web层中对[Reactive programming](https://www.reactivemanifesto.org/)的支持。它为服务端创建响应式web应用、也为客户端消费REST服务提供支持。

本文中，我会演示一个采用Spring WebFlux的示例web应用。Spring 5+中的WebFlux支持两种不同编程风格：传统注解风格和新的函数式风格。本文中，我将坚持传统的注解风格，并尝试在另一篇博客中详细描述类似的应用程序，但是以函数式风格定义endpoints。我的重点将是编程模型。


## 数据和服务层

如下是一个相当简单的REST接口，它支持一个酒店资源的CRUD操作：

```java
public class Hotel {
    private UUID id;
    private String name;
    private String address;
    private String state;
    private String zip;
    ....
}
```

示例中采用Cassandra存储这个entity，并且使用[Spring Data Cassandra](https://spring.io/blog/2016/11/28/going-reactive-with-spring-data)对响应式编程的支持使数据层获得响应式特性，从而可以支持这样的API。我采用两个repositories，一个便于上面酒店entity的存储，另一个维护可以通过酒店第一个字母进行检索的冗余数据。

```java
public interface HotelRepository  {
    Mono<Hotel> save(Hotel hotel);
    Mono<Hotel> update(Hotel hotel);
    Mono<Hotel> findOne(UUID hotelId);
    Mono<Boolean> delete(UUID hotelId);
    Flux<Hotel> findByState(String state);
}
public interface HotelByLetterRepository {
    Flux<HotelByLetter> findByFirstLetter(String letter);
    Mono<HotelByLetter> save(HotelByLetter hotelByLetter);
    Mono<Boolean> delete(HotelByLetterKey hotelByLetterKey);
}
```


返回一个entity实例的操作现在返回一个[Mono](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Mono.html)类型，返回多个entity实例的则返回一个[Flux](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Flux.html)类型。

鉴于此，我简单介绍一种返回响应类型的方法：当更新一个酒店实体时，我必须先删除HotelByLetter存储库维护的冗余数据，然后重新创建一个。这可以通过使用Flux和Mono类型提供的优秀运算符来完成，如下所示：

```java
public Mono<Hotel> update(Hotel hotel) {
    return this.hotelRepository.findOne(hotel.getId())
            .flatMap(existingHotel ->
                    this.hotelByLetterRepository.delete(new HotelByLetter(existingHotel).getHotelByLetterKey())
                            .then(this.hotelByLetterRepository.save(new HotelByLetter(hotel)))
                            .then(this.hotelRepository.update(hotel))).next();
}
```


## Web层

现在回到文章的焦点：支持Web层中基于注解的响应式编程模型支持！

多年来，[@Controller](https://docs.spring.io/spring/docs/5.0.x/javadoc-api/org/springframework/stereotype/Controller.html)和[@RestController](https://docs.spring.io/spring/docs/5.0.x/javadoc-api/org/springframework/web/bind/annotation/RestController.html)注解一直是Spring MVC的REST endpoint支持的重要组成部分。传统上，他们已经使用多时并返回java POJO。响应模型中的这些控制器已经被调整用来接收和返回Reactive类型——在我的例子中是Mono和Flux——但是另外也支持Rx-java和[Reactive Streams](http://www.reactive-streams.org/)类型。

鉴于此，我的Controller如下所示：

```java
@RestController
@RequestMapping("/hotels")
public class HotelController {
    ....
    @GetMapping(path = "/{id}")
    public Mono<Hotel> get(@PathVariable("id") UUID uuid) {
        return this.hotelService.findOne(uuid);
    }
    @PostMapping
    public Mono<ResponseEntity<Hotel>> save(@RequestBody Hotel hotel) {
        return this.hotelService.save(hotel)
                .map(savedHotel -> new ResponseEntity<>(savedHotel, HttpStatus.CREATED));
    }
    @PutMapping
    public Mono<ResponseEntity<Hotel>> update(@RequestBody Hotel hotel) {
        return this.hotelService.update(hotel)
                .map(savedHotel -> new ResponseEntity<>(savedHotel, HttpStatus.CREATED))
                .defaultIfEmpty(new ResponseEntity<>(HttpStatus.NOT_FOUND));
    }
    @DeleteMapping(path = "/{id}")
    public Mono<ResponseEntity<String>> delete(
            @PathVariable("id") UUID uuid) {
        return this.hotelService.delete(uuid).map((Boolean status) ->
                new ResponseEntity<>("Deleted", HttpStatus.ACCEPTED));
    }
    @GetMapping(path = "/startingwith/{letter}")
    public Flux<HotelByLetter> findHotelsWithLetter(
            @PathVariable("letter") String letter) {
        return this.hotelService.findHotelsStartingWith(letter);
    }
    @GetMapping(path = "/fromstate/{state}")
    public Flux<Hotel> findHotelsInState(
            @PathVariable("state") String state) {
        return this.hotelService.findHotelsInState(state);
    }
}
```

传统的@RequestMapping、@GetMapping和@PostMapping并没有被改变，变得是返回类型。比如，预期只有一个结果的，现在返回一个Mono类型，之前预期返回一个列表的，现在则返回一个Flux类型。

通过使用Spring Data Cassandra中的响应式支持，整个Web到服务到后端设置都是响应式的，特别针对本文的关注点，非常直观易见。

简单地尝试这篇文章后面的代码可能会更易于理解，你可以在我的[GitHub仓库](https://github.com/bijukunjummen/sample-webflux-annot-cassandra)中获得这些代码。


