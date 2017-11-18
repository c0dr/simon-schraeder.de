---
title: "Better UI: Display passwords"
date: 2013-09-12T19:50:20+01:00
draft: false
---

Signup for one service and one common thing you'll see is a masked password box. This provides security because bystanders can't read the password. It comes with a downside though: Your user will be annoyed if he makes a typo. This is especially annoying if the password needs to be typed two times to prevent typos and you have to type both passwords again.

But there is a simple solution to this problem: Provide an "unmask" function. 

All you need to do is to change the type of the field from password to text.

To demonstrate this, I'll be using bootstrap and jQuery, but it can obviously be done with plain javascript and without any framework.

The checkbox will be appended to the password field to make it look good and easy for the user to understand.


`<div class="form-group">
  <label class="col-md-4 control-label" for="password">Password</label>
  <div class="col-md-4">
    <div class="input-group">
      <span class="input-group-addon">     
          <input type="checkbox" class="showPassword"><label>Show</label>     
      </span>
      <input id="password" name="password" class="form-control" type="password" placeholder="" required="">
    </div>
  </div>
</div>`


To make this usable you just need to add a simple handler that changes the type of it on click.

`<script>
$('.showPassword').on('change', function () {
    var cache = $(this).parent().next();
    if (cache.attr('type') == 'text') {
        cache.attr('type', 'password')
    } else {
        cache.attr('type', 'text')
    }
});
</script>`

It doesn't work on IE <9 and I haven't split tested it yet, so use it with caution. 

<strong>Demo</strong>:
<iframe width="100%" height="300" src="http://jsfiddle.net/cNC2K/embedded/result%2Cjs%2Chtml/" allowfullscreen="allowfullscreen" frameborder="0"></iframe>

Related to this: Please don't prevent pasting of passwords (at least at the signin page): It provides no security at all and discourages user to use a password manager, which could mean they use insecure passwords.

And if your security certificate requires this, you might think about getting rid of it because it provides no value at all.

<blockquote class="twitter-tweet" lang="en"><p><a href="https://twitter.com/passy">@passy</a> We&#39;d lose our security certificate if we allowed pasting. It could leave us open to a &quot;brute force&quot; attack. Thanks ^Steve</p>&mdash; British Gas Help (@BritishGasHelp) <a href="https://twitter.com/BritishGasHelp/statuses/463619139220021248">May 6, 2014</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>