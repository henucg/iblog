---
layout: page
title: Category
permalink: /category
---

<script>
  var key = decodeURI(location.pathname);
  key = key.substring('/category/'.length);
  if (key.indexOf('/') != -1) {
    key = '';
  }

  var href = decodeURI(location.href);
  var category = href.split("=");
</script>

正在查看 "<script>document.write(category[1]);</script>" 下的文章

<div class="post">
  <div class="post-archive">
  {% for post in site.posts %}
    <!-- <h2>{{ post.date | date: "%Y" }}</h2> -->
    <ul class="listing" style="display: none;">
      <li>
      <span class="date">{{ post.date | date: "%Y/%m/%d" }}</span>
      <a href="{{ post.url | prepend: site.baseurl }}" key="{{ post.categories }}" target="_blank">
      {% if post.title %}
  		{{ post.title }}
  	  {% else %}
  		{{ site.page_no_title }}
  	  {% endif %}
  	  </a>
  	</li>
    </ul>
  {% endfor %}
  </div>
</div>

<script>
  window.onload=function() {
    var items = $('.post-archive a');
    for (var i=0; i<items.length; i++) {
      var item = items[i];
      if ($(item).attr('key').toLowerCase().indexOf(category[1].toLowerCase()) == -1) {
        $(item.parentElement.parentElement).remove();
      } else {
      	$(item.parentElement.parentElement).show();
      }
    }
    if ($('.post-archive a').length == 0) {
      $('.post-archive').html('<font color="red">没有记录</font>')
    }
  }
</script>
