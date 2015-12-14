---
layout: post
category : javascript
tagline: "Helpful snippets for nightwatch js tests"
tags : [ nightwatch, javascript, phantomjs, selenium, behaviour driven development ]
---

```javascript
this.featuredArticleFollowThrough = function() {
    var mainFeaturedArticleTitleLink = '#mini-panel-static_bars > div > div > div > div > div > div > div > div.views-row.views-row-1.views-row-odd.views-row-first > div.views-field.views-field-nothing > span > div > div:nth-child(2) > a';
    //console.log('---Checking featured article link matches destination article title.---');
    browser
      .verify.cssClassPresent(mainFeaturedArticleTitleLink, 'node-title', 'Featured article link is present on front page.')
      .getText(mainFeaturedArticleTitleLink, function(response) {
        var featuredArticleTitle = response.value;
        this
          .click(mainFeaturedArticleTitleLink)
            .waitForElementVisible('.node-title', 90000, 'Article title is present.')
            .verify.containsText('.node-title', featuredArticleTitle, 'Article title text matches originating link on frontpage.')
       })
    return browser;
  }
```
