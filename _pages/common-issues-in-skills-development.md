---
ID: 32446
post_title: Common issues in Skills development
author: Kathy Reid
post_excerpt: ""
layout: page
permalink: >
  http://mycroft.ai/documentation/skills/common-issues-in-skills-development/
published: true
post_date: 2017-12-03 06:24:16
---
# Common issues in **Skills** development

## I've added new phrases in the .voc file, but Mycroft isn't recognizing them

Compound words like "don't", "won't", "shouldn't" etc. are normalized by Mycroft - so they become "do not", "will not", "should not". You should use the normalized words in your `.voc` files.

@TODO I need more information on issues that Skill developers have