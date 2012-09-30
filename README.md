# Ajaxify Rails

No more full page reloads for your Rails app! Yay! 

Automatically makes your app loading content in the background via Ajax.

Works by turning all internal links into Ajax links that trigger an update of the page's content area. 
Also form submissions are automatically turned into Ajax requests.

Features: 

- Uses the html5 history interface for changing the url and making the browser's back and forward buttons work with Ajax.
- Falls back to a hash based approach for browsers without the history interface (like Internet Explorer version <10)
- Hash based and non-hash URLs are interchangeable.
- Transparently handles redirects and supports page titles and flash messages.
- Tested with Chrome, Firefox, Safari and Internet Explorer 8+

Demo: http://ajaxify-demo.herokuapp.com/ (the first page load might take a while, as heroku needs to spin up a dyno)

Blog Post: http://rubyandrails.posterous.com/introducing-the-ajaxify-rails-gem

Inspired by the pjax_rails gem (https://github.com/rails/pjax_rails).

## Requirements

- Ruby 1.9 and the asset pipeline.
- Your app doesn't use named anchors (#). Named anchors can't be correctly represented in the fallback hash based url scheme.

## Installation

Add this line to your application's Gemfile:

    gem 'ajaxify_rails'

And then execute:

    $ bundle

In your application.js file add:

    //= require ajaxify_rails

## Usage

Call `Ajaxify.init()` in your layout's javascript.
Do this as early as possible to ensure Ajaxify's interchangeable url schemes (history api vs. hash based urls)
work most effectively. 

The later you call `init()`, the later potential redirects from one scheme to another are performed,
which means the more unnecessary work the browser has to do.


### Content Area

Ajaxify assumes that your app has a content container html tag with the id `main`.
This tag is the container wrapping the yield statement in your layout.
If yield doesn't have a wrapper in your app yet, you need to supply one to get ajaxification working:

    #main
      = yield

You can change the content wrapper in your javascript by setting

    Ajaxify.content_container = 'content_container_id'
    
    
### Loader Animation

You probably like to have a loader image to be displayed to the user while content loads via Ajax.
This is simple. Ajaxify automatically inserts a loader div with the class `ajaxify_loader` into
the content wrapper before starting an Ajax request. So just supply styles for `.ajaxify_loader` in your css, with an
animated gif as a background.
    

### Page Title

If you define a method called `page_title` in your application controller, Ajaxify will automatically
update the page's title tag after the main content has changed.

### Navigation and other Layout Updates

It's a common use case to have a navigation that needs to change its appearence and possibly functioning when the user navigates
to a different section of the page. Ajaxify provides a success callback that is triggered after successful
updates of the page's main content. Just hook into it in your javascript and make your layout changes:

    Ajaxify.success ->
      # update navigation and/or other layout elements


### Flash Messages

Ajaxify correctly displays your flash messages after ajaxified requests. To do so it stores them in cookies.
By default, only `flash[:notice]` is supported. If you are using for example `flash[:warning]` as well you have to set:

    Ajaxify.flash_types = ['notice', 'warning']
    
Also make sure that you supply invisible wrapper tags in your layout for each flash type you use, with the id set to the type, e.g.:

    #notice{ style: "#{'display:none' unless flash[:notice]}" }
      = flash[:notice] 
    
### Links or Forms that need to trigger full Page Reloads

We all know them. Those big requests changing the layout of the page so significantly that 
loading ajax into a content area and doing some minor layout tweaks here and there simply doesn't cut it. Sigh.

There might also be links and forms which already have their own ajax functionality.

Well, to turn Ajaxify off for certain links and forms, simply add the class `no_ajaxify` directly to the link or form:

    = link_to 'Change everything!', re_render_it_all_path, class: 'no_ajaxify'


### Root Redirects

Sometimes you need to redirect on the root url. 

For example you might have a localized application with the locale inside the url.
When a user navigates to `your_domain.com` he/she gets redirected to e.g. `your_domain.com/en/`. This works fine in browsers supporting
the html 5 history api. However, for browsers without the history api like Internet Explorer before version 10, Ajaxify needs hints
about your url structure to not get confused (it creates endless redirects otherwise!). You need to explicitly supply all possible root
paths.

Example: if your app's root url potentially redirects to `your_domain.com/en/` and `your_domain.com/de/`
you need to hint Ajaxyfiy like this:

    Ajaxify.base_paths = ['de', 'en']

Important: `Ajaxify.base_paths` need to be set before `Ajaxify.init()` is called!


### Extra Content

Sometimes you need to do non trivial modifications of the layout whenever the content in the main content area of your site changes.
Ajaxify allows you to attach arbitrary html to ajaxified requests. This extra html is then stripped from the main content
that is inserted in the content area. But before that a callback is triggered which can be used to grab the extra content and do something with it.
To use this feature you need to provide a method `ajaxify_extra_content` in your ApplicationController:

    def ajaxify_extra_content
      ... your extra html ...
    end

For example you could provide url for a widget in the layout like this:

    def ajaxify_extra_content
      "<div id='my_fancy_widget_html'> some html </div>"
    end

And then, on the client side hook into Ajaxify using the `handle_extra_content` callback and select the widget html via `#ajaxify_content`:

    Ajaxify.handle_extra_content = ->
      $('#my_fancy_widget').html $('#ajaxify_content #my_fancy_widget_html').html()


### Reference: All Javascript Options and Callbacks

Here is a reference of all options and callbacks you can set on the client side via `Ajaxify.<i>option_or_callback</i> =` :

    Option/Callback        Default      Description

    active                 true         Toggles link ajaxification.
    content_container     'main'        Id of the container to insert the main content into ("yield wrapper").
    base_paths             null         Base path segments for applications with root url redirects.

    on_before_load         null         Callback: Called before the ajaxify request is started.
    on_success             null         Callback: Called when an ajaxify requests finished successfully.
    on_success_once        null         Callback: Like on_success but only called once.
    handle_extra_content   null         Callback: Called before extra content is stripped from the ajax request's response.

    flash_types            ['notice']   Flash types your Rails app uses. E.g. ['notice', 'warning', 'error']
    flash_effect           null         Callback: Called for each flash type after flash is set.
    clear_flash_effect     null         Callback: Called for each flash type whenever flash message is not present

Also check the example app source code for usage: https://github.com/ncri/ajaxify_rails_demo_app


## Contributing

1. Fork it
2. Create your feature branch (`git checkout -b my-new-feature`)
3. Commit your changes (`git commit -am 'Added some feature'`)
4. Push to the branch (`git push origin my-new-feature`)
5. Create new Pull Request