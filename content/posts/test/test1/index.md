---
date: 2017-04-09T10:58:08-04:00
description: "sfdsdfadf asdfadfad dadfasd"
featured_image: "/images/Pope-Edouard-de-Beaumont-1844.jpg"
images: ["/images/Pope-Edouard-de-Beaumont-1844.jpg"]
tags: ["scene"]
categories: "Story"
title: "Индукционные дайсы"
draft: true
---

# Идея

```go
func getCookie(name string, r interface{}) (*http.Cookie, error) {
	rd := r.(*http.Request)
	cookie, err := rd.Cookie(name)
	if err != nil {
		return nil, err
	}
	return cookie, nil
}
func setCookie(cookie *http.Cookie, w interface{}) error {
	// Get write interface registered using `Acquire` method in handlers.
	wr := w.(http.ResponseWriter)
	http.SetCookie(wr, cookie)
	return nil
}
```

```yaml
- name: playbook
  hosts:
    all

  roles:
    - role: ping

    - role: gitlab
      become: true
      become_user: root
      tags:
        - gitlab

    # - role: requirements
    #   become: true
    #   become_user: root

```

Купить чтото типа этого (мелкие светодиоды, которые припянны к мелкой же катушке, в комплекте идет катушка питания и драйвер, для работы)
https://www.aliexpress.com/item/4001032411407.html  
https://www.aliexpress.com/item/1005001465709865.html  
https://www.aliexpress.com/item/1005003657171960.html  
https://www.aliexpress.com/item/1005003986112404.html  

и этого (молды дайсов для заливки прозрачным эпоксидом)
https://www.aliexpress.com/item/4000529598566.html 
https://www.aliexpress.com/item/4000519544606.html  
https://www.aliexpress.com/item/1005001580843011.html  
https://www.aliexpress.com/item/4000480237645.html  
https://www.aliexpress.com/item/4001049811216.html  
https://www.aliexpress.com/item/4000576263901.html  


Цель - получить набор кубиков, который при броске в катушку будет начинать светиться
Цель 2 - замаскировать управляющую катушку в поле для бросков, а то и в мешочек для костей с местом для батарейки и выключателем.
Цель 3 - сделать свою систему управления позволяющую сделать мерцание или например не жарить катушку на всю если нет птребителей

https://www.aliexpress.com/item/4000480237645.html  
https://www.aliexpress.com/item/1005001627792955.html  

Примерный тех процесс

Заливаем молд прозрачной эпоксидкой
помещаем в еще застывающую эпоксидку такой мини светодиод
каким то образом не даем ему опуститься на одну из сторон укбика пока эпокситдка застывает
(альтернатива - заливаенм часть кубика и даем ей застыть - помущаем светодлиод и заливаем остаток, не так красиво да, если захочеться дымчатый или прозрачный)
для самом\го простого варианта мпосле застывания эпоксидки  заполняем символы чемто леаким образом внутренние стенки будут непрозрачными и орнганеизуют чтот типа зеркала. Поиадее даже немного запоротый дас в такой конфиграции должен светиться достаточно ярко. После высыхания лака герметик извлекается отставляя прозрачный акрил на местах символов.

для определения дайсов в котушке можно давать короткие импульсы и смотреть за нпряжением.токосм через катушку. если ток возрос при импульсе илинпряжение просело то скорее всего потребитель в катушке (или батарейка села, ноэто мониторить отдельно) и нужно давать длительный импульс. Как только ток.напряжение выровнялись - дайсы скорее всего подняли и можно переходить в режим сканирования и не жарить постоянно,