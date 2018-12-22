+++
title = "Path Traversal on Retrofit"
date = 2018-11-24T13:15:43+10:00
draft = false
toc = false
categories = ["breaking"]
tags = ["path traversal", "cve", "injection"]
+++

> **Update**: CVE created [CVE-2018-1000850](https://nvd.nist.gov/vuln/detail/CVE-2018-1000850). For some reason, the wording on the CVE is quite confusing. That is not what I submitted. Hope they fix it soon.

[Retrofit](https://github.com/square/retrofit/) is a type-safe HTTP client for Android and Java developed by Square, Inc. It is a very popular library with over 30k starts in Github and more than 120 contributors.

I found a path traversal vulnerability when using `encoded=true` on `@Path` parameters. Below is an unit test reproducing the issue. This test was added on `RequestFactoryTest` class (it is not upstream though).

<!--more-->

{{< highlight java  >}}

@Test public void getWithEncodedPathSegmentsSecurity() {

    class Example {
        @GET("/foo/bar/{ping}/") //
            Call<ResponseBody> method(@Path(value = "ping", encoded = true) String ping) {
            return null;
        }
    }

    Request request = buildRequest(Example.class, "../baz/pong/more");
    assertThat(request.method()).isEqualTo("GET");
    assertThat(request.headers().size()).isZero();
    assertThat(request.url().toString()).isEqualTo("http://example.com/foo/baz/pong/more/");
    assertThat(request.body()).isNull();
}

{{< / highlight >}}


Given the type safety provided by the library, the issue does not affect GET requests that much. If the results returned by the path traversal are different from what was the original code was designed to parse, it will occur a deserialisation issue. For example, if the original URL was `/repository/{id}` and an attacker sends an id like `../user/1` the URL requested by retrofit will be `/user/1`. When parsing the response, Retrofit will be expecting a data structure mapping a repository not a user. Therefore, causing a deserialisation issue.


However, `POST`, `PUT`, `DELETE`s and any requests that change the server will be affected. The deserialisation happens only after getting the response. The HTTP request at this point would have been completed and would have applied the intending effect caused by the path traversal. In the example above, if the request was a `DELETE` rather than a `GET` the attacker would have been successful in send a `DELETE` request on a user rather than a repository.


`POST` and `PUT` are not so easily exploitable, they are more dependable on the context of how they were designed. If there are two similar REST resources in the same API, it is perfectly possible to use one of the endpoints to request to `POST` or `PUT` on a different (but similar) resource.


Square has fixed the issue [here](https://github.com/square/retrofit/commit/b9a7f6ad72073ddd40254c0058710e87a073047d). Square also has released the fix on version 2.5.0. The vulnerability was introduce in a [very early commit](https://github.com/square/retrofit/commit/01d6fb1a133cea6fac18f2ef1ecab1cbee86b569). Which means all versions bellow 2.5.0 are vulnerable.