---
title : "Agregar Nota A Tiddlywiki"
date : "2025-07-31"
image : ""
tags : []
categories : [pamplinas]
description : ""
---


[Tiddlywiki](https://tiddlywiki.com/) es una aplicación genial para guardar notas en un solo archivo html. Gracias a su api puedes añadir notas usando este commando `curl`

```sh
curl -X PUT -i 'http://192.168.0.12:8080/recipes/default/tiddlers/NewTiddlerTitle' --data '{
	 "tags": "firstTag anotherTag",
	 "creator": "gene",
	 "modifier": "gene",
	 "text": "The use of knowledge in society"
}' -H "X-Requested-With: TiddlyWiki"

