---
title: ASP.NET MVC3/MVC4 Unobtrusive Validation with Twitter Bootstrap style views
categories:
- hacks
tags:
- ASP.NET
- ASP.NET MVC
- control group
- jQuery Unobtrusive Validation
- jQuery Validation
- Twitter Bootstrap
- Unobtrusive Validation
---

If you're working with ASP.NET MVC3/MVC4 and you're opting to enhance your views with [Twitter Bootstrap](http://twitter.github.com/bootstrap/)  front-end toolkit, which is an excellent tool  for rapidly developing web applications, you may encounter some obstacles in your way while enabling client side unobtrusive validation. As you may know this feature is achieve through  the co-op of two JavaScript plugins: [jQuery Validation](http://docs.jquery.com/Plugins/Validation) and  [MVC 3 Unobtrusive jQuery Validation](http://nraykov.wordpress.com/2011/06/06/asp-net-mvc-3-unobtrusive-client-side-validation/).

**The real stuff.**

This is what you wanna get:

![Desired Client Validation Behavior]({{ site.baseurl }}/images/capture1.png){:class="img-responsive"}

**The code**

In order to achieve that you may follow a couple of ways, but with the exception of modifying the CSS styles of the MVC 3 Unobtrusive jQuery Validation Plugin, most of them lead to the same end: modify the highlight/unhighlight handlers to obtain the desire effect. This is easy, but  first you have to layout the view using the `.control-group` and `.controls` style classes as the following example shows:

![Code Snipet]({{ site.baseurl }}/images/capture21.png){:class="img-responsive"}

Once did this for every form field you should proceed to write some jQuery code to ensure the label get the right style because the ASP.NET MVC3 Html.LabelFor helper wont let you add custom HTML attributes. So..

``` javascript
$("#register-form .control-group>label").addClass('control-label');
```

Of course you may also write a custom Html helper to works out did for you.

Now you must code some JavaScript  to overwrite the default behavior of the jQuery Validation plugin in matters of highlight/highlight forms error:

``` javascript
function boostrapHighlight(element, errorClass, validClass) {
     $(element).closest(".control-group").addClass("error");
     $(element).trigger('highlated');
};

function boostrapUnhighlight(element, errorClass, validClass) {
     $(element).closest(".control-group").removeClass("error");
     $(element).trigger('unhighlated');
};

$.validator.setDefaults({
     ignore: "",
     highlight: function (element, errorClass, validClass) {
          boostrapHighlight(element, errorClass, validClass);
     },
     unhighlight: function (element, errorClass, validClass) {
          boostrapUnhighlight(element, errorClass, validClass);
     }
});

```

Voila!  it's done!. Now when the plugin desires to highlight an element it looks for the closest `.control-group` element and decorates it with the `.error` CSS class achieving the desire bootstrap style validation behavior. The unhighlight works similarly but in reverse.

**Further explanation**

In the previous code fragment there are a couple of stuff you may wonder why I did it.

The first, after highlighting every element I trigger an event named `highlithed` and the same thing after unhighlighting which unleash the `unhighlithed` event. This statement allows me to triggers some particular behavior in the element which allows me for example: highlight the tab with errors in the form show in the [image](#desired). I got to say I've tried a cleaner approach to trigger customs behavior when a highlight occurs by using a Publisher-Subscriber Pattern to subscribe custom callbacks when these functions(highlight/unhighlight) were called but it turns out in a asynchronous mess when you a couple of forms in the same page. So in the end I prefer to use jQuery [custom events](http://dailyjs.com/2009/11/14/jquery-custom-events/) which provide a built in means to use the publish subscribe pattern in a way that is functionally equivalent.

The second, when I override the defaults of the jQuery Validation Plugin I set the ignore property to none(""). This allows the validation to ignore none DOM elements and highlight/unhighlight even the hidden elements.

**Conclusions**

The integration of Twitter Bootstrap front end toolkit is an excellent choice to make cleaner layout of the view. Today we have learned that tweak the MVC 3 Unobtrusive jQuery Validation Plugin to work with Bootstrap validation styles it's really easy.
