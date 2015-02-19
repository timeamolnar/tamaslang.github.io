---
layout: post
title: Lightweight model mapping with java 8
---

In a multi tier application you might need certain abstraction regarding your data.

It is better to limit the number of such layers, not to go overboard with mappings, abstractions.<br/>
We all have seen some examples where some couldn't find a good balance and developers ended up spending most of their times writing mappings.

But having too many layers/abstractions usually result wasting hours on "get and set" and what is worse as a result of mapping significant number of the bugs will be about certain data not displayed/saved.

The following image from <a href="http://geek-and-poke.com/geekandpoke/2013/7/13/foodprints" target="_blank">geek &amp; poke</a> describes the problem well.
![placeholder](./assets/footprints2.jpg "Good Architect leaves a footprint")

A good trade-off can be that you have a data domain layer, e.g. JPA annotated data entities and a DTO layer which is how you represent the data to the outside world.<br/>
Data domain can have multiple DTO representation depending the functionality.

Let's say you have a book inventory app. In the Data domain you have a Book entity which has many properties and has some relations to let's say authors, categories, etc.
You might want an endpoint where you would like to list all the books with only title and author name.
In this case certain mapping will be necessary, and here is a Java8 example to doing it a nice/lightweight way:

## mapper function
```java
private Function<Book,BookDTO> bookToBookDTO = new Function<Book, BookDTO>() {
    public BookDTO apply(Book book) { return new BookDTO(book.getIsbn(), book.getTitle(),book.getAuthor());}
};
```

## usage:
```java
// Map one:
bookToBookDTO.apply(aBookEntity);
```
```java
// Map a list:
aBookEntityList.stream().map(bookToBookDTO).collect(Collectors.toList());
```

In certain cases libraries like <a href="http://dozer.sourceforge.net/" target ="_blank">Dozer</a> or <a href="http://modelmapper.org/" target="_blank">modelmapper</a> can be helpful; the second one
will be introduced in an upcoming post.

T.