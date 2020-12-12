---
title: 'Rails 5.2 with webpacker, bootstrap, stimulus starter'
tags:
  - rails webpack stimulus javascript bootstrap
categories: Program
date: 2018-05-02 09:19:32
---


#### Create Project

```bash
# Last few parameters(--skip-* part) is only my habbit not actully required
$ rails new <project_name> --webpack=stimulus --database=postgresql --skip-coffee --skip-test
$ cd <project_name>
$ rails db:create
```

<!--more-->

#### Configure scss architecture

If you are using some front end framework you may like to integrate stylesheet into components with webpack or you just like to integrate stylesheets with webapck like me. This is a way that we integrate that into **webpacker**.

>  NOTE: This is only the convention of our team you can avoid this step and keep stylesheet in assets/.

```bash
$ mkdir app/javascript/stylesheets
$ touch app/javascript/stylesheets/application.scss
$ touch app/javascript/stylesheets/_variables.scss
$ touch app/javascript/stylesheets/_base.scss
```

After create files please write down styles as follow:

_app/javascript/stylesheets/application.scss_

```scss
@import 'variables';
@import 'base';
```

_app/javascript/stylesheets/\_variables.scss_

```scss
$colors: (
  major: #00D252,
  minor: #2F3B59
);
```

 _app/javascript/stylesheets/\_base.scss_

```scss
h1 {
  color: map-get($colors, major);
}
```

On the top of _app/javascript/packs/application.js_

```js
import 'stylesheets/application'
```



#### （Optional）Integrate stimulus manually

If you are not use `--webpack=stimulus` for create project or install `stimulus` in existed project.

```bash
$ yarn add stimulus
$ mkdir app/javascript/controllers
# To provide a example for testing stimulus
$ touch app/javascript/controllers/clipboard_controller.js
```

#### （Optional）Configure stimulus

_app/javascript/s/packs/application.js_

```js
/* eslint no-console:0 */
// This file is automatically compiled by Webpack, along with any other files
// present in this directory. You're encouraged to place your actual application logic in
// a relevant structure within app/javascript and only use these pack files to reference
// that code so it'll be compiled.
//
// To reference this file, add <%= javascript_pack_tag 'application' %> to the appropriate
// layout file, like app/views/layouts/application.html.erb
import 'stylesheets/application'

import { Application } from "stimulus"
import { definitionsFromContext } from "stimulus/webpack-helpers"

const application = Application.start()
// The path you may like to change to under `pack` that path will be `./controllers`
// but convention will be in `/app/javascript/controllers`
const context = require.context("controllers", true, /\.js$/)
application.load(definitionsFromContext(context))
```

Example of testing `stimulus`:

_app/javascript/controllers/clipboard_controller.js_

```js
import { Controller } from 'stimulus'

export default class extends Controller {
  static targets = ['source']
  initialize() {
    console.log('clipboard initialize')
  }
  connect() {
    console.log('clipboard connect')
    if (document.queryCommandSupported('copy')) {
      this.element.classList.add('clipboard--supported')
    }
  }
  copy(e) {
    e.preventDefault()
    this.sourceTarget.select()
    document.execCommand('copy')
  }
}
```

#### Create a example controller and view

```bash
$ rails g controller pages example
```

Add _app/views/pages/example.html.erb_

```html
<h1>Hello, World</h1>
<hr>
<div data-controller="clipboard members dashboard">
  PIN
  <input type="text" data-target="clipboard.source" value="1234" readonly>
  <button data-action="clipboard#copy" class="clipboard-button">
    Copy to Clipboard
  </button>
</div>
```

#### Add pack to layout

Open _app/views/layout/application.html.erb_ then add `pack tags` to `<head>`

```html
<%= stylesheet_pack_tag 'application' %>
<%= javascript_pack_tag 'application' %>
```

#### Add route

_config/routes.rb_

```rb
Rails.application.routes.draw do
  # For details on the DSL available within this file, see http://guides.rubyonrails.org/routing.html
  root 'pages#example'
end
```

Then you can test

```bash
$ rails s
```

Navigate to `localhost:3000` should see as follow

![](https://imgur.com/Khh0VY6.png)

Until here you should complete Rails 5.2 using webpacker with stimulus and stylesheets.

For common practical stiuation you may want to use bootstrap v4.x.

#### Install bootstrap

```bash
# https://getbootstrap.com/docs/4.1/getting-started/webpack/
$ yarn add jquery popper.js bootstrap
```

#### Import boostrap stylesheets

In _app/javascript/stylesheets/application.scss_ add bootstrap

```scss
@import '~bootstrap/scss/bootstrap';
@import 'variables';
@import 'base';
```

#### imiport bootstrap JavaScript

_app/javascript/packs/application.js_

```js
import 'bootstrap'
```

#### Configure webpacker

Add configuration to _config/webpack/environment.js_. If you do not setup this step, the abilities related to `Popper.js` such as `tooltip` will not working.

```js
const { environment } = require('@rails/webpacker')
const webpack = require('webpack')
/**
 * Automatically load modules instead of having to import or require them everywhere.
 * Support by webpack. To get more information:
 *
 * https://webpack.js.org/plugins/provide-plugin/
 * http://j.mp/2JzG1Dm
 */
environment.plugins.prepend(
  'Provide',
  new webpack.ProvidePlugin({
    $: 'jquery',
    jQuery: 'jquery',
    jquery: 'jquery',
    'window.jQuery': 'jquery',
    Popper: ['popper.js', 'default']
  })
)
module.exports = environment
```

Sometimes you may like to use jQuery in views you should expose jQuery to global

#### expose jQuery to global for views

```bash
# https://webpack.js.org/loaders/expose-loader/
$ yarn add expose-loader -D
```

Add configuration to _config/webpack/environment.js_

```js
/**
 * To use jQuery in views
 */
environment.loaders.append('expose', {
  test: require.resolve('jquery'),
  use: [{
    loader: 'expose-loader',
    options: '$'
  }]
})
```

#### Other convention of our team

```bash
$ mkdir -p lib/templates/active_record/model
$ touch lib/templates/active_record/model/model.rb
```

_lib/templates/active_record/model/model.rb_

```rb
<% module_namespacing do -%>
class <%= class_name %> < <%= parent_class_name.classify %>
  # scope macros

  # Concerns macros

  # Constants

  # Attributes related macros
<% if attributes.any?(&:password_digest?) -%>
  has_secure_password
<% end -%>

  # association macros
<% attributes.select(&:reference?).each do |attribute| -%>
  belongs_to :<%= attribute.name %><%= ', polymorphic: true' if attribute.polymorphic? %>
<% end -%>

  # validation macros

  # callbacks

  # other

  private
    # callback methods
end
<% end -%>
```
