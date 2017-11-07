---
ID: 32452
post_title: Adding a new language to Mycroft
author: Kathy Reid
post_excerpt: ""
layout: page
permalink: http://mycroft.ai/?page_id=32452
published: false
---
# Adding a new language to Mycroft

## Introduction

The Mycroft software suite has many elements. Adding language support for Mycroft means that each of those elements has to support the new language. This page walks you what has to be done for each of the elements.

## Speech to text (STT)

Speech to text (STT) is the part of Mycroft that translates spoken words into text. That text is then used by intent parsers, and then by **Skills**.

In order to support a new language, a Speech to Text engine (STT engine) must be available.

STT engines are made available by different vendors, and they each have different licenses and usage restrictions.

_NOTE: As of late 2017, Mycroft.AI is working with Mozilla Voice to build an open source STT engine for multiple languages. This development will occur over 2018_

* [List of languages supported by Google STT] (https://stackoverflow.com/questions/14257598/what-are-language-codes-in-chromes-implementation-of-the-html5-speech-recogniti)
* [List of languages supported by IBM Watson Bluemix](https://www.ibm.com/watson/developercloud/speech-to-text/api/v1/#sessionless_methods
* [List of languages supported by Wit.AI](https://wit.ai/faq)

_NOTE: PocketSphinx is not currently supported on Mycroft_

@TODO Once I've found a STT engine for my language, what do I do with it?

## Text to Speech