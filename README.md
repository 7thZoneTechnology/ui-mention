# ui-mention
Facebook-like @mentions for text inputs built around composability

## Contribute

0. `npm install`
0. `npm install -g gulp bower`
0. `bower install`
0. `gulp [watch]`
0. Compiling the example code: `gulp example [watch]`

## Usage

For now, you should create a child-directive to customize (API probably going to change)

```js
.directive('myMention', function($http){
  return {
    require: 'uiMention',
    link($scope, $element, $attrs, uiMention) {
      /**
       * Converts a choice object to a human-readable string
       *
       * @param  {mixed|object} choice The choice to be rendered
       * @return {string}              Human-readable string version of choice
       */
       uiMention.label = function(choice) {
         return `${choice.first_name} ${choice.last_name}`;
       };

      /**
       * Retrieves choices
       *
       * @param  {regex.exec()} match    The trigger-text regex match object
       * @return {array[choice]|Promise} The list of possible choices
       */
      uiMention.findChoices = function(match, mentions) {
        return $http.get(...).then(...);
      };
    }
  };
});
```
You have to build the HTML yourself:
```html
<div class="ui-mention-container">
  <textarea ng-model="data" ui-mention my-mention></textarea>
  <div class="ui-mention-highlight"></div>
</div>
<ul class=dropdown" ng-if="$mention.choices.length">
  <li ng-repeat="choice in $mention.choices"
    ng-class="{active:$mention.activeChoice==choice}"
    ng-click="$mention.select(choice)">
    {{::choice.first_name}} {{::choice.last_name}}
  </li>
</ul>
```
And the CSS:
```scss
.ui-mention-container {
  position: relative;
  [ui-mention] {
    min-height: 100px;
    background: none;
    position: relative;
    z-index: 2;
    box-sizing: content-box; // Prevent scrollbar for autogrow
  }
  .ui-mention-highlight {
      white-space: pre-wrap;
      position: absolute;
      top: 0;
      bottom: 0;
      left: 0;
      right: 0;
      background: white;
      color: rgba(0,0,0,0);
      z-index: 1;
      span {
        border-radius: 2px;
        background-color: lightblue;
        border: 1px solid blue;
        padding: 0 2px;
        margin: -1px -3px;
      }
    }
  }
}
```

## Amazing Features!

_All these features come at the amazingly low price of DO IT YOURSELF and $0.00. YMMV._

Find things!:
```js
mention.findChoices = function(match) {
  // Matches items from search query
  return [/* choices */]
    .filter( choice => ~this.label(choice).indexOf(match[1]) );
}
```

Type too damn fast? Throttle that bitch:
```js
mention.findChoices = _.throttle(function(match) {
  return [/* choices */];
}, 300);
```

Hate redundancy? De-dupe that shiz:
```js
mention.findChoices = function(match, mentions) {
  return [ /* choices */ ]
    .filter( choice => !mentions.some( mention => mention.id === choice.id ) )
};
```

Use the awesome power of the internet:
```js
mention.findChoices = function(match) {
  return $http.get('/users', { params: { q: match[1] } })
    .then( response => response.data );
}
```

Your servers are slow? Bitch please.
```js
mention.findChoices = function(match) {
  mention.loading = true;
  return $http.get(...)
    .then( response => {
      mention.loading = false;
      return response.data;
    });
}
```

Dropdown that list like it's hot:
```html
<ul ng-if="$mention.choices.length" class="dropdown">
  <li ng-repeat="choice in choice" ng-click="$mention.select(choice)">
    {{::choice.name}}
  </li>
</ul>
```

SPINNIES!
```html
<ul ng-if="$mention.choices.length" class="dropdown">
  <li ng-show="$mention.loading">Hacking the gibson...</li>
  <li ng-repeat=...>...</li>
</ul>
```
