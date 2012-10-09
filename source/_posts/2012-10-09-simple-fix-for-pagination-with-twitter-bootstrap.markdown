---
layout: post
title: "Simple fix for pagination with Twitter Bootstrap"
date: 2012-10-09 16:40
comments: true
categories: 
- Rails
- Tips
---

As a Rails guy, I always perform my table pagination using mislav's [will_paginate](https://github.com/mislav/will_paginate) gem. However, when I use it combined with Twitter Bootstrap, I get an undesired result:

{% img /images/posts/pagination-bad.png %}

There is a very simple fix for this which doesn't require using an [additional gem](https://github.com/yrgoldteeth/bootstrap-will_paginate).

Add the following CSS to your application:

{% codeblock assets/stylesheets/pagination_fix.css lang:css %}
    //-------------------------------------------------------------------------------
    //   Pagination fix for will_paginate and bootstrap
    //-------------------------------------------------------------------------------

    .pagination {
      span.disabled {
        color: #aaa !important;
      }
      em.current {
        float: left;
        padding: 0 14px;
        line-height: 38px;
        text-decoration: none;
        background-color: white;
        border: 1px solid #DDD;
        border-left-width: 0;
      }
      .previous_page {
        border-left: 1px solid #ddd;
      }
    }
{% endcodeblock %}

Refresh your page and the pagination issues should be resolved:

{% img /images/posts/pagination-good.png %}

Hope this helps!
