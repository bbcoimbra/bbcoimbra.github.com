---
layout: post
title: "[Postgres Tricks] Intervalos variáveis"
---

    select current_timestamp + ( 2 || ' days')::interval;

