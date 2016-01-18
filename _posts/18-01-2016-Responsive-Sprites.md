---
layout: post
category : css
tagline: "Responsive sprites without using css background property"
tags : [ css, html, background, sprites, responsive]
---

![Responsive sprites without background property]({{ site.url }}/assets/sprite_social.png)

```html
<li class="social">
        <a class="stretchy" href="https://www.facebook.com/robbreport" target="_blank">
          <img class="spacer" alt="icon" src="/sites/all/themes/at_robbreport/img/sprite_spacer.png">
          <img class="sprite facebook" alt="robb report facebook" src="/sites/all/themes/at_robbreport/img/sprite_social.png">
        </a>
        <a class="stretchy" href="https://twitter.com/robbreport" target="_blank">
          <img class="spacer" alt="icon" src="/sites/all/themes/at_robbreport/img/sprite_spacer.png">
          <img class="sprite twitter" alt="robb report twitter" src="/sites/all/themes/at_robbreport/img/sprite_social.png">
        </a>
        <a class="stretchy" href="https://instagram.com/robbreport" target="_blank">
          <img class="spacer" alt="icon" src="/sites/all/themes/at_robbreport/img/sprite_spacer.png">
          <img class="sprite instagram" alt="robb report instagram" src="/sites/all/themes/at_robbreport/img/sprite_social.png">
        </a>
        <a class="stretchy" href="https://www.pinterest.com/robbreport/" target="_blank">
          <img class="spacer" alt="icon" src="/sites/all/themes/at_robbreport/img/sprite_spacer.png">
          <img class="sprite pinterest" alt="robb report pintrest" src="/sites/all/themes/at_robbreport/img/sprite_social.png">
        </a>
</li>
```

```css
.at_robbreport #block-rr-header-rr-header ul.header_links li.social a {
    display: inline-block;
    width: 19%;
    margin-right: 3%;
    max-width: 32px;
}
.stretchy {
    position:relative;
    overflow:hidden;
    max-width:32px;
    max-height: 32px;
}
.stretchy .spacer {
    width: 100%;
    height: auto;
    display: block;
}
.stretchy .sprite {
    position:absolute;
    top:0;
    display: block;
    left:0;
    max-width:none;
    height:292%;
}
.stretchy .sprite.facebook {
}
.stretchy .sprite.twitter {
    left:-200%;
}
.stretchy .sprite.instagram {
    left:-400%;
}
.stretchy .sprite.pinterest {
    left:-600%;
}
```
  
Notes:
Taken from: http://tobyj.net/responsive-sprites/
*  sprite_spacer.png is a 1x1 transparent png - get yours here: http://www.1x1px.me/
*  Added value: The tobyj responsive sprite demo does not support a master sprite image that is irregular in height to the target sprite dimensions.
*  Sprites being targetted are those on the top row. Other images in the sprite are not relavant here, except to illustrate a typical master sprite with irregular width etc...
*  The top row spacing is: 32px square icon, 32px padding, 32px icon, 32px padding etc... this let's us use a nice 100%, 200% left offset later.
*  Master sprite dimension: 566px x 93px
*  Master sprite height as percentage of 32px target dimension: 290.525% hence .sprite {height: 292%;}
*  Live at: www.robbreport.com

