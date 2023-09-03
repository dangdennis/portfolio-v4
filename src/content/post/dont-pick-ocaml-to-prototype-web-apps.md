---
title: Don't pick OCaml to prototype web apps
description: Failing to build web apps for fun and profit
publishDate: 09-03-2023
draft: false
---

I wanted to build all my new projects in OCaml. It'd be a welcomed change from
the world of Typescript and Python.

I've been a novice hanging around the
[OCaml community](https://discuss.ocaml.org/) for the past few years. The
discussions around algebraic effects and promising
[language features](https://discuss.ocaml.org/t/next-priority-for-ocaml/12561/110)
appeal to me. I eagerly daydream that this is the language and ecosystem I can
set myself to mastering for sheer fun! Sure, there'll be some pain, but I can
overcome it!

I never reached the promiseland.

I build for the web. I don't write compilers, or interface with low-level linux
sys calls. The requirements for a mature web stack is just tremendous. You need
the ssl stack, the database stack, the http stack, the html stack, and so much
more. There are heroes attempting to make OCaml For The Web a reality but unless
you're Jane Street, Ahrefs, or any of the OCaml consultancy groups, it's just a
slow grindfest to solo build a database-based web application compared to the
JS/Python/.NET/JVM world. Well duh! Fortunately Chatgpt-4 has helped me overcome
many a issues, like writing a [Caqti](https://github.com/paurkedal/ocaml-caqti)
query to select, insert, update my postgres database.

Check out the Caqti code below that executes a vector search with Postgres's
`pg_embedding`, and returns a list of tuples of results. It's not really the
simplest code. [ppx_rapper](https://github.com/roddyyaga/ppx_rapper) exists to
make writing caqti code easier, but when you compare it to Typescript's Prisma
or Drizzle, or .NET's EntityFramework or Dapper, life is a bit harder.

```
open Caqti_request.Infix
open Caqti_type.Std
open Core

let similar_bookmarks ~db:(module Db : Caqti_lwt.CONNECTION) ~(embeddings : string list) =
  let stringified_embeddings = embeddings |> String.concat ~sep:"," in
  let embedding_sql_arg = "{" ^ stringified_embeddings ^ "}" in
  let open Caqti_type in
  let web_bookmark_typ =
    let encode w = Ok (w.id, w.title, w.url) in
    let decode (id, title, url) = Ok { id; title; url } in
    custom ~encode ~decode (tup3 int string string)
  in
  let query =
    (Caqti_type.string ->* web_bookmark_typ)
    @@ "  SELECT * FROM web_bookmark ORDER BY embedding <-> ? LIMIT 5;"
  in
  Db.collect_list query embedding_sql_arg |> App_error.map_caqti_error
;;
```

I'll fast forward to make sure I'm not just ranting. I tried to help out by
triaging and fixing bits and pieces on the
[Dream](https://github.com/aantron/dream) web framework, but with such a small
community, enhancements to libraries just take way longer.

So while I love OCaml, Dune, Opam, and its principles so far, I'll have to set
this one aside whenever I want to build quick web apps. What are my other
choices? I don't really know. All I do know is that I still want to pick a fun
programming language. Half the fun is the tool. I flip-flop between Rescript,
F#, or just plain old Typescript and Python. Or the occasional attempts at Rust.
