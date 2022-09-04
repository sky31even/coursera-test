# Module 5 的一个小tips
> 09/04/2022 最后一个作业花了我一点时间来理解，记在这里，将来也许对其他人有用。
> 课程链接：[HTML, CSS, and Javascript for Web Developers - Johns Hopkins University](https://www.coursera.org/learn/html-css-javascript-for-web-developers)


这里是 `homHtmlUrl` 获取的 `html snippet`：

```html
    <div class="jumbotron">
      <img src="images/jumbotron_768.jpg" alt="Picture of restaurant" class="img-responsive visible-xs">
    </div>

    <div id="home-tiles" class="row">
      <div class="col-md-4 col-sm-6 col-xs-12">
        <a href="#" onclick="$dc.loadMenuCategories();"><div id="menu-tile"><span>menu</span></div></a>
      </div>
      <div class="col-md-4 col-sm-6 col-xs-12">
      <!-- 下面这行是重点区域 -->
        <a href="#" onclick="$dc.loadMenuItems({{randomCategoryShortName}});">
          <div id="specials-tile"><span>specials</span></div>
        </a>
      </div>
    </div><!-- End of #home-tiles -->
```

这里是对应的 `js` 文件片段：

```js
function buildAndShowHomeHTML (categories) {

  // Load home snippet page
  $ajaxUtils.sendGetRequest(
    homeHtmlUrl,
    function (homeHtml) {
      var chosenCategoryShortName = chooseRandomCategory(categories).short_name;
	      // 下面这行是重点区域
      var homeHtmlToInsertIntoMainPage = insertProperty(homeHtml, "randomCategoryShortName", "'"+chosenCategoryShortName+"'");
      insertHtml("#main-content", homeHtmlToInsertIntoMainPage);
    },
    false); 
}

dc.loadMenuItems = function (categoryShort) {
  showLoading("#main-content");
  $ajaxUtils.sendGetRequest(
    menuItemsUrl + categoryShort,
    buildAndShowMenuItemsHTML);
};
```

最开始写到 `js` 重点区域那一行时，我认为这个 `chosenCategoryShortName` 变量反正已经是字符串了，即使作业里面提示了要添加 surrounding 但我还是无视了，因为我看到下面的 `loadMenuItems` 方法就是直接用加号连接的，也就是两个字符串的相接，我认为应该不会有问题。

但是在实际运行中，始终在提示我  `L is not defined` （L 也可能是随机获取到的其他 category 的 short name），于是我回溯代码多次，发现了问题所在：
- 在创建 `homeHtmlToInsertIntoMainPage` 这个变量的时候，使用的 `insertProperty` 这个方法，返回的是一整个 html 的字符串，因此 `chosenCategoryShortName` 如果不带单引号，也会被转换成一个变量名，这时候`loadMenuItems` 其实还尚未调用；
- 在加载完成后，因为 `onclick` 事件而调用  `loadMenuItems`方法时，整个页面的 html 其实是下面这样，注意 `loadMenuItems`这个方法的参数现在不会被识别为字符串，而是被当作一个变量输入，于是查询整个 stack 发现没有这个变量，从而报错。

```html
    <div class="jumbotron">
      <img src="images/jumbotron_768.jpg" alt="Picture of restaurant" class="img-responsive visible-xs">
    </div>

    <div id="home-tiles" class="row">
      <div class="col-md-4 col-sm-6 col-xs-12">
        <a href="#" onclick="$dc.loadMenuCategories();"><div id="menu-tile"><span>menu</span></div></a>
      </div>
      <div class="col-md-4 col-sm-6 col-xs-12">
      <!-- 下面这行是重点区域 -->
        <a href="#" onclick="$dc.loadMenuItems(L);">
          <div id="specials-tile"><span>specials</span></div>
        </a>
      </div>
    </div><!-- End of #home-tiles -->
```