# Survey Tools

This Choregraphe package provides a skeleton for creating interactive demos and surveys using the Pepper robot and its chest-mounted tablet. This package makes use of HTML/Javascript to create visual displays while also providing access the animation and speech capabilities of the robot.

## Setup

To install this package on a robot, open the Survey.pml file in Choregraphe. Then use the upload to robot feature to send the package to the robot. This will add a launcher entry on the tablet of the robot which when selected will cause the robot to display the `html/index.html` file on the tablet.

*Note: only one survey service is capable of running on the robot at one time. To install multiple surveys you will either need to rename the service in `service.py` and in `html/javascripts/main.js` or rmeove any old surveys.*

## Basic Usage

The following HTML code, which we will discuss in depth below, is an example of one of the most basic interactive applications that can be generated with the survey system.

```html
<html>
  <head>
    <link rel="stylesheet" type="text/css" href="styles/main.css?v=201808300902" />
    <script src="/libs/qimessaging/2/qimessaging.js"></script>
    <script src="javascripts/polyfill.js"></script>
  </head>
  <body>
    <div class="content">
    <form method="post" autocomplete="off" novalidate>
      <div class="survey-group">
        <div id="start" class="survey-box">
          <button type="button" class="btn-navigate" data-next="0">Begin</button>
        </div>
        <div id="0" class="survey-box">
          <h1>Hello, my name is Pepper</h1>
          <button type="button" class="btn-submit">Finish</button>
        </div>
      </div>
      <div class="popup">
        <span class="popuptext" id="invalid-popup">
          <span style="color: red">&#10071;</span>
          <span id="invalid-message"></span>
        </span>
      </div>
    </form>
    </div>
    <footer id="progress-bar" class="progress-bar"></footer>
    <script src="javascripts/main.js?v=201808300902"></script>
  </body>
</html>
```

While much of the above example is general setup, the two main organisational components of the survey that we are interested in are the elements annotated with the `survey-group` and the `survey-box` class names. We will now provide a description of these two components, as well as some of their children.

### Survey Group

The `survey-group` class name indicates that a container will container one or more `survey-box` containers. A `survey-group` will become automatically active when one of its `survey-box` children become active. This class is a convenience class for organising large surveys, and does not affect the overall flow of the survey.

### Survey Box

The `survey-box` class name indicates that the container to which it is applied is a stand-alone *page* of the survey. Through-out the course of the survey, only one `survey-box` is visible at any given time.

```html
<div id="start" class="survey-box">
</div>
```

**Important:** Any container using the `survey-box` also requires an `id` attribute for navigation as seen in the example above.

### Navigation

Navigating between `survey-box` containers is achieved using simple buttons annotated with the `btn-navigate` class. Navigation can either be a simple linear progression through a set of questions, or involve complex branching depending on your needs.

#### Simple Navigation

Linear navigating between `survey-box` can be achieved using a simple HTML button with the attached `btn-navigate` class such as in the example below:

```html
<button type="button" class="btn-navigate" data-next="0">Begin</button>
```

The destination of these buttons is determined by the `data-next` attribute, which will find a survey-box with an field `id` matching its value.

#### Branching Navigation

The following code snippet highlights an example of using a branching button to navigate to one of two different survey-box containers depending on user input.

```html
<div id="start" class="survey-box">
  <h1>How are you feeling?</h1>
  <input type="radio" id="option-1" name="Happy" />
  <input type="radio" id="option-2" name="Unhappy" />

  <button type="button"
          class="btn-navigate"
          data-next="option-1=happy-box;option-2=sad-box;">
    Begin
  </button>

</div>
<div id="happy-box" class="survey-box">
</div>
<div id="happy-box" class="survey-box">
</div>
```

The important piece of this example is the button (which we will describe in detail below):

```html
<button type="button"
        class="btn-navigate"
        data-next="option-1=happy-box;option-2=sad-box">
  Begin
</button>
```

Like a simple navigation button, the branching button must have the class `btn-navigate` applied to it to mark it as a navigation button. Like the simple button, it also requires a `data-next` attribute to determine where to transition to.

The value of this `data-next` attribute differs however in that it provides multiple possible transitions options, with each option being seperated by a `;` symbol. Each option is comprised at most two parts: a condition check; and an id of a target `survey-box`.

Note that the final option does not need a condition, and can be regarded as the default option if no other options have a satisfied condition. This allows us to re-write the `data-next` attribute of the above example as `data-next="option-1=happy-box;sad-box"` which is more succint.

When the button is clicked, the system will check each transition option in left to right order, and as soon as it finds one where a condition is satisied, it will transition to the indicated target `survey-box`.

The value for these conditions can either be an radio/checkbodx input element id, or a javascript function call. For an input element id, the condition will be satisfied if the indicated element is currently selected. For a function call, indicated by `call:my_function` (Example: `data-next="call:test=happy-box;sad-box;"`), the condition will be satisfied if the function returns true.

### Dialog

For every `survey-group`, excepting the initially displayed box, the robot can be made to speak to the user to provide additional information or instructions.

To get the robot to speak, simply provide a script between a set of `<h1></h1>` tags within a `survey-group`. The system will detect this and send the text inside the tags to the speech engine of the robot.

```html
<div id="0" class="survey-box">
  <h1>Hello, my name is Pepper</h1>
</div>
```

Note that the robot will only read out the first `<h1></h1>` tag within a `survey-box`, and that while the robot is speaking, nothing will be shown on the tablet to encourage the user to look at the face of the robot.

#### Animated Dialog

Animated dialog can be achieved by applying the `data-animated` attribute to the `<h1>` tag as seen below:

```html
<h1 data-animated>I will read this with contextualised animations</h1>
```

By default the robot will read dialog using the contextual dialog animation engine. However, this can be adjusted by specifing a value for the `data-animated` tag from the set `disabled|random|contextual`.

Additionally, NAOqi speech engine annotations can be used within the `<h1>` tag to provide explicit instructions to the speech engine, as seen below (with a user-friendly `h1` which will be displayed when the dialog has completed):

```html
<h1 data-animated class="hidden">^run(animations/Stand/Waiting/WakeUp_1) Well \Pau=1000\ That was interesting.</h1>
<h1>Text to display to the user</h1>
```

A description of available text annotations can be found within the [NAOQi Documentation](http://doc.aldebaran.com/2-4/naoqi/audio/alanimatedspeech.html#annotated-text)

#### Automatic Transitions

Often you will want the robot to read some dialog, before automatically progressing on to another `survey-box`. To achieve this, annotate the `<h1>` tag with a `data-auto-transition` attribute as see below:

```html
<h1 data-auto-transition>The robot will not read this aloud</h1>
```

*The survey-box must contain a `btn-navigation` button that the system will auto-click to transition to the next `survey-box`.*


#### Displaying Content While Reading

To display content, such as images or text while the robot is reading, annotate the `survey-box` tag with a `data-show-dialog` attribute as seen below:

```html
<div id="0" class="survey-box" data-show-dialog>
  <h1>Hello, my name is Pepper</h1>
</div>
```

This can be used in conjunction with auto-transitions to provide an automated slide-show to the user.

#### Blocking Reading

In the event that you do not want the robot to read out anything on a particular `survey-box` simply annotate the first `<h1>` tag with `data-ignore`, such as in the following example:

```html
<h1 data-ignore>The robot will not read this aloud</h1>
```


### Debugging

As developer tools are not accessible on the tablet attached to the Pepper robot it is often difficult to diagnose issues in your HTML/Javascript. We recommend adding the following code snippet inside the `<head></head>` tags of your HTML document:

```html
<script>
  window.addEventListener("error", handleError, true);
  function handleError(evt) {
    if (evt.message) { // Chrome sometimes provides this
        alert("error: "+evt.message +" at linenumber: "+evt.lineno+" of file: "+evt.filename);
    } else {
        alert("error: "+evt.type+" from element: "+(evt.srcElement || evt.target));
    }
  }
</script>
```

This code snippet will generate a pop-up window when an error occurs allowing for easier debugging of issues.

## Examples

Examples can be found in the `html` folder of this repository.