---
layout: page
title: Captcha breaking
description: Solving captchas faster than humans
img: assets/img/captcha/closesquare.jpg
importance: 3
category: school
related_publications: false
---

For my highschool graduation project I created an algorithm that solved the SecurImage CAPTCHA faster than humans. 

This project is about my highschool graduation project. It was written in 2017 when I was 17.
The goal was to create a python library that is capable of breaking the [SecurImage CAPTCHA](https://www.phpcaptcha.org/).
The code in [this repo](https://github.com/DavidEncrypted/captcha) is capable of generating the right answer to the captcha about 1 in 7 attempts. This is very usable as the securimage captcha instantly generates a new attempt. The code is also very simple and thus very quick.

---

The graduation project report can be downloaded [here](/assets/pdf/pws_david_schep.pdf). Note: It is in Dutch.

---

I also wrote an example usage of the python library [at this repo](https://github.com/DavidEncrypted/alphabreak). Here I used the library to spam an illegal dark web market place (sells for example drugs and weapons) with fake accounts. This was made to show the value that my library could have for law enforcement.

---

To show off my project I had to put on a stand at the infomarket. Here I let people battle my algorithm to decide, once and for all, who was the better CAPTCHA solver.

<div class="row mt-2">
    <div class="col-sm mt-2 mt-md-0">
        {% include figure.liquid path="assets/img/captcha/close.jpg" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
    <div class="col-sm mt-2 mt-md-0">
        {% include figure.liquid path="assets/img/captcha/far.jpg" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
</div>

---

It was not close, the algorithm only lost once.
