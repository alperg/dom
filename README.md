# JavaScript DOM traversal and manipulation Student guide

## DOM ready and window load

```js
// $(document).ready(function() {...});
document.addEventListener('DOMContentLoaded', function() {...});

// $(window).load(function() {...});
window.addEventListener('load', function() {...});
window.onload = function() {...};
```

## Selectors

```js
// var divs = $('ul.nav > li');
var items = document.querySelectorAll('ul.nav > li');
var firstItem = document.querySelector('ul.nav > li');

// var title = $('#title');
var title = document.getElementById('title');

// var images = $('.image');
var images = document.getElementsByClassName('image');

// var articles = $('article');
var articles = document.getElementsByTagName('article');
```

| Selector | Returns a collection | Returns a LIVE collection | Return type | Built-in forEach | Works with any root element |
|----------|----------------------|---------------------------|-------------|------------------|-----------------------------|
| `getElementById` | 🚫 | N/A | Reference to an `Element` object or `null` | N/A | 🚫 |
| `getElementsByClassName` | ✅ | ✅ | `HTMLCollection` | 🚫 | ✅ |
| `getElementsByTagName` | ✅ | ✅ | `HTMLCollection` according to the spec (`NodeList` in WebKit) (*) | 🚫 | ✅ |
| `querySelector` | 🚫 | N/A | `Element` object or `null` | N/A | ✅ |
| `querySelectorAll` | ✅ | 🚫 | Static `NodeList` of `Element` objects | ✅ | ✅ |
  
(*) *The [latest W3C specification](https://dvcs.w3.org/hg/domcore/raw-file/tip/Overview.html) says it returns an `HTMLCollection`; however, this method returns a [`NodeList`](https://developer.mozilla.org/en-US/docs/Web/API/NodeList) in WebKit browsers. See [bug 14869](https://bugzilla.mozilla.org/show_bug.cgi?id=14869) for details.*

Since all of the selectors (except `getElementById`) support querying a node other than `document`, finding results trivial:

```js
// $(el).find(selector);
el.querySelectorAll(selector);
```

## Creating elements

```js
// var newDiv = $('<div>');
var newDiv = document.createElement('div');

// There is no equivalent in jQuery for createTextNode.
// You can always use the DOM method, or write a jQuery wrapper around it.
// The closest thing you may be able to find is when creating new elements,
// you can specify the text part separately.
// var newTextNode = $('<div>', { text: 'hello world' });
var newTextNode = document.createTextNode('hello world');
```

## Adding elements to the DOM

```js
// $(parent).append(el);
parent.appendChild(el);

// $(parent).prepend(el);
parent.prepend(el);

// $(parent).prepend(el); 
parent.insertBefore(el, parent.firstChild);
el.insertBefore(node);

// $(el).before(htmlString);
el.insertAdjacentHTML('beforebegin', htmlString);

// $(el).after(htmlString);
el.insertAdjacentHTML('afterend', htmlString);
```

## Traversing the DOM

```js
// $(el).children();
el.children // only HTMLElements
el.childNodes // includes comments and text nodes
// You can't `forEach` through `children` unless you turn it into an array first.

// $(el).parent();
el.parentNode

// $(el).closest(selector);
el.closest(selector);

// $(el).first();
el.firstElementChild; // only HTMLElements
el.firstChild; // includes comments and text nodes

// $(el).last();
el.lastElementChild; // only HTMLElements
el.lastChild; // includes comments and text nodes

// First and last alternative
var nodeList = document.querySelectorAll('.some-class');
var first = nodeList[0];
var last = nodeList[nodeList.length - 1];

// $(el).siblings();
[].filter.call(el.parentNode.children, function(child) {
  return child !== el;
});

// $(el).prev();
el.previousElementSibling; // only HTMLElements
el.previousSibling; // includes comments and text nodes

// $(el).next();
el.nextElementSibling; // only HTMLElements
node.nextSibling; // includes comments and text nodes

// $.contains(el, child);
el !== child && el.contains(child);
```

## Traversing a node list

```js
var nodes = document.querySelectorAll('.class-name');

// 1.
var elements = Array.prototype.slice.call(nodes);
elements.forEach(noop);

// 2. (clean, but creates a new array)
[].forEach.call(nodes, noop);

// 3.
Array.prototype.forEach.call(nodes, noop);
```

## Closest

Find the closest element that matches the target selector:

```js
var node = document.getElementById('my-id');
var isFound = false;

while (node instanceof Element) {
  if (node.matches('.target-class')) {
    isFound = true;
    break;
  }
  node = node.parentNode;
}
```

## Removing and replacing nodes

```js
// $(el).remove();
el.parentNode.removeChild(el);
el.remove();

// Remove all GIF images from the page
[].forEach.call(document.querySelectorAll('img'), function (img) {
  if (/\.gif/i.test(img.src)) {
    img.remove();
  }
});

// $(el).replaceWith($('.first'));
el.parentNode.replaceChild(newNode, el);

// $(el).replaceWith(string);
el.outerHTML = string;
```

## Cloning nodes

```js
// $(el).clone();
el.cloneNode(true);
```

Pass in `true` to also clone child nodes.

## Checking if a node is empty

```js
// $(el).is(':empty');
!el.hasChildNodes();
```

## Emptying an element

```js
// $(el).empty();
var el = document.getElementById('el');

// This is faster
while (el.firstChild) {
  el.removeChild(el.firstChild);
}
```

You could also do `el.innerHTML = ''`, but would be slower than the above method.

## Comparing elements

```js
// $(el).is($(otherEl));
el === otherEl

// $(el).is('.my-class');
el.matches('.my-class');

// $(el).is('a');
el.matches('a');
```

## Getting and setting text content

```js
// $(el).text();
el.textContent

// $(el).text(string);
el.textContent = string;
```

There's also `innerText` and `outerText`:

* `innerText` was non-standard, while `textContent` was standardised earlier.
* `innerText` returns the visible text contained in a node, while `textContent` returns the full text. For example, on the following element: `<span>Hello <span style="display: none;">World</span></span>`, `innerText` will return 'Hello', while `textContent` will return 'Hello World'. As a result, `innerText` is much more performance-heavy: it requires layout information to return the result.

Here is the official warning for `innerText`: *This feature is non-standard and is not on a standards track. Do not use it on production sites facing the Web: it will not work for every user. There may also be large incompatibilities between implementations and the behavior may change in the future.*

## Getting and setting outer/inner HTML

```js
// $('<div>').append($(el).clone()).html();
el.outerHTML;

// $(el).replaceWith(string);
el.outerHTML = string;

// $(el).html();
el.innerHTML

// $(el).html(string);
el.innerHTML = string;

// $(el).empty();
el.innerHTML = '';
```

## Getting and setting attributes

```js
// $(el).attr('tabindex');
el.getAttribute('tabindex');

// $(el).attr('tabindex', 3);
el.setAttribute('tabindex', 3);
```
  
Since elements are just objects, most of the times we can directly access (and set) their properties:

```js
var oldId = el.id;
el.id = 'foo';
```

Some other properties we can access directly:

```js
node.style;
node.class;
node.href;
node.checked;
node.disabled;
node.selected;
```

For data attributes we can either use `el.getAttribute('data-something')` or resort to the `dataset` object:

```js
// $(el).data('camelCaseValue');
string = element.dataset.camelCaseValue;

// $(el).data('camelCaseValue', 'foo');
element.dataset.camelCaseValue = 'foo';
```

## Styling elements

```js
// $(el).css('background-color', '#3cca5e');
el.style.backgroundColor = '#3cca5e';

// $(el).hide();
el.style.display = 'none';

// $(el).show();
el.style.display = '';
```

To get the values of all CSS properties for an element you should use `window.getComputedStyle(element)` instead:

```js
// $(el).css(ruleName);
getComputedStyle(el)[ruleName];
```

## Working with CSS classes

```js
// $(el).addClass('foo');
el.classList.add('foo');

// $(el).removeClass('foo');
el.classList.remove('foo');

// $(el).toggleClass('foo');
el.classList.toggle('foo');

// $(el).hasClass('foo');
el.classList.contains('foo');
```

## Getting the position of an element

```js
// $(el).outerHeight();
el.offsetHeight

// $(el).outerWidth();
el.offsetWidth

// $(el).position();
{ left: el.offsetLeft, top: el.offsetTop }

// $(el).offset();
var rect = el.getBoundingClientRect();
{
  top: rect.top + document.body.scrollTop,
  left: rect.left + document.body.scrollLeft
}
```

## Binding events

```js
// $(el).on(eventName, eventHandler);
el.addEventListener(eventName, eventHandler);

// $(el).off(eventName, eventHandler);
el.removeEventListener(eventName, eventHandler);
```

If working with a collection of elements, you can bind an event handler to each one of them by using a loop:

```js
// $('a').on(eventName, eventHandler);
var links = document.querySelectorAll('a');
[].forEach.call(links, function (link) {
  link.addEventListener(eventName, eventHandler);
});
```

## Event delegation

Can add to higher element and use 'matches' to see if specific child was clicked (similar to jQuery's `.on`):

```js
// $('ul').on('click', 'li > a', eventHandler); // Pay attention to the second argument here
var el = document.querySelector('ul');
el.addEventListener('click', event => {
  if (event.target.matches('li')) {
    // event handling logic
  }
});
```

## The event object

```js
var node = document.getElementById('my-node');
var onClick = function (event) {
  // this = element

  // can filter by target = event delegation
  if (!event.target.matches('.tab-header')) {
    return;
  }

  // stop the default browser behaviour
  event.preventDefault();

  // stop the event from bubbling up the dom
  event.stopPropagation();

  // other listeners on this node will not fire
  event.stopImmediatePropagation();
};

node.addEventListener('click', onClick);
node.removeEventListener('click', onClick);
```

## Animations

```js
// $(el).fadeIn();
function fadeIn(el) {
  el.style.opacity = 0;

  var last = +new Date();
  var tick = function () {
    el.style.opacity = +el.style.opacity + (new Date() - last) / 400;
    last = +new Date();

    if (+el.style.opacity < 1) {
      (window.requestAnimationFrame && requestAnimationFrame(tick)) || setTimeout(tick, 16);
    }
  };

  tick();
}
```

Or, if you are only supporting IE10+:

```js
el.classList.add('show');
el.classList.remove('hide');
```

```css
.show {
  transition: opacity 400ms;
}

.hide {
  opacity: 0;
}
```

## Looping over and filtering through collections of DOM elements

Note this is not necessary if you are using `querySelector(All)`.

```js
// $(selector).each(function (index, element) { ... });
var elements = document.getElementsByClassName(selector);

[].forEach.call(elements, function (element, index, arr) { ... });

//or
Array.prototype.forEach.call(elements, function (element, index, array) { ... });

// or
Array.from(elements).forEach((element, index, arr) => { ... }); // ES6
```

Same concept applies to filtering:

```js
// $(selector).filter(":even");
var elements = document.getElementsByClassName(selector);

[].filter.call(elements, function (element, index, arr) {
  return index % 2 === 0;
});
```

Recall that `:even` and `:odd` use 0-based indexing.

Another filtering example:

```js
var nodeList = document.getElementsByClassName('my-class');
var filtered = Array.prototype.filter.call(nodeList, function (item) {
  return item.innerText.indexOf('Item') !== -1;
});
```

## Stringify & Parse

```js
JSON.stringify(object);

// $.parseJSON(string);
JSON.parse(string);

// $.trim(string);
string.trim();
```
