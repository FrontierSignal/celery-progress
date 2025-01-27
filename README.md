# Celery Progress Bars for Django

Drop in, dependency-free progress bars for your Django/Celery applications.

Super simple setup. Lots of customization available.

## Demo

[Celery Progress Bar demo on Build With Django](https://buildwithdjango.com/projects/celery-progress/)

## Installation

```bash
pip install celery-progress
```

## Usage

### Prerequisites

First add `celery_progress` to your `INSTALLED_APPS` in `settings.py`.

Then add the following url config to your main `urls.py`:

```python
url(r'^celery-progress/', include('celery_progress.urls')),  # the endpoint is configurable
```

### Recording Progress

In your task you should add something like this:

```python
from celery import shared_task
from celery_progress.backend import ProgressRecorder
import time

@shared_task(bind=True)
def my_task(self, seconds):
    progress_recorder = ProgressRecorder(self)
    for i in range(seconds):
        time.sleep(1)
        progress_recorder.set_progress(i + 1, seconds)
    return 'done'
```

### Displaying progress

In the view where you call the task you need to get the task ID like so:

**views.py**
```python
def progress_view(request):
    result = my_task.delay(10)
    return render(request, 'display_progress.html', context={'task_id': result.task_id})
```

Then in the page you want to show the progress bar you just do the following.

#### Add the following HTML wherever you want your progress bar to appear:

**display_progress.html**
```html
<div class='progress-wrapper'>
  <div id='progress-bar' class='progress-bar' style="background-color: #68a9ef; width: 0%;">&nbsp;</div>
</div>
<div id="progress-bar-message">Waiting for progress to start...</div>
```

#### Import the javascript file.

**display_progress.html**
```html
<script src="{% static 'celery_progress/celery_progress.js' %}"></script>
```

#### Initialize the progress bar:

```javascript
// vanilla JS version
document.addEventListener("DOMContentLoaded", function () {
  var progressUrl = "{% url 'celery_progress:task_status' task_id %}";
  CeleryProgressBar.initProgressBar(progressUrl);
});
```

or

```javascript
// JQuery
$(function () {
  var progressUrl = "{% url 'celery_progress:task_status' task_id %}";
  CeleryProgressBar.initProgressBar(progressUrl)
});
```

## Customization

The `initProgressBar` function takes an optional object of options. The following options are supported:

| Option | What it does | Default Value |
|--------|--------------|---------------|
| pollInterval | How frequently to poll for progress (in milliseconds) | 500 |
| progressBarId | Override the ID used for the progress bar | 'progress-bar' |
| progressBarMessageId | Override the ID used for the progress bar message | 'progress-bar-message' |
| progressBarElement | Override the *element* used for the progress bar. If specified, progressBarId will be ignored. | document.getElementById(progressBarId) |
| progressBarMessageElement | Override the *element* used for the progress bar message. If specified, progressBarMessageId will be ignored. | document.getElementById(progressBarMessageId) |
| onProgress | function to call when progress is updated | CeleryProgressBar.onProgressDefault |
| onSuccess | function to call when progress successfully completes | CeleryProgressBar.onSuccessDefault |
| onError | function to call when progress completes with an error | CeleryProgressBar.onErrorDefault |
