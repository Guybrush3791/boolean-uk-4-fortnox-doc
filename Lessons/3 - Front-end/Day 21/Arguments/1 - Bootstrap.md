# Bootstrap
[Bootstrap](https://getbootstrap.com/) is a popular front-end toolkit with ready-made CSS classes and components for building responsive, mobile-first web pages quickly. Its optional JavaScript supports interactive interface elements.

<div class="iframe-container"> <iframe src="https://getbootstrap.com/" frameborder="0" allowfullscreen></iframe> </div>

## What it offers
- Consistent styles for typography, forms, buttons, tables, navigation and other common elements, reducing the amount of custom CSS needed.
- A 12-column grid and layout utilities for responsive designs across different screen sizes.
- Optional JavaScript components such as modals, tooltips, dropdowns and carousels that integrate with the CSS.

## How to start
- Load Bootstrap from a CDN by linking its CSS in the page head and its JavaScript bundle before the closing body tag. Include the responsive `viewport` meta tag for correct mobile scaling.
- Structure the page with containers, rows and columns, then use utility classes for spacing, colours and alignment.

## Why it is useful
- Speeds development from prototype to production with reusable, customisable components and utilities.
- Uses a mobile-first approach so layouts adapt to phones, tablets and desktops with minimal extra work.

## Documentation
The best way to learn a front-end framework is usually through its [official documentation](https://getbootstrap.com/docs).

## Layout and grid

```html
<!-- Container and grid -->
<div class="container">
  <div class="row">
    <div class="col-12 col-md-8">Main</div>
    <div class="col-12 col-md-4">Sidebar</div>
  </div>
</div>
```

```html
<!-- Responsive columns and gutters -->
<div class="container">
  <div class="row g-3">
    <div class="col-sm-6 col-lg-3">A</div>
    <div class="col-sm-6 col-lg-3">B</div>
    <div class="col-sm-6 col-lg-3">C</div>
    <div class="col-sm-6 col-lg-3">D</div>
  </div>
</div>
```


## Spacing helpers

```html
<!-- Margin (m) and padding (p): t,r,b,l,x,y; scale 0-5 -->
<div class="mt-3 mb-4 px-3 py-2">Spacing utilities</div>
```


## Display and visibility

```html
<!-- Display utilities -->
<div class="d-none d-md-block">Visible ≥ md</div>
<div class="d-block d-md-none">Hidden ≥ md</div>
```


## Flexbox utilities

```html
<!-- Flex rows, alignment, gap -->
<div class="d-flex align-items-center justify-content-between gap-3">
  <span>A</span><span>B</span><span>C</span>
</div>
```


## Typography

```html
<!-- Headings and lead -->
<h1 class="h3">Semantic h1 styled as h3</h1>
<p class="lead mb-0">Lead paragraph for emphasis.</p>
<small class="text-muted">Muted note</small>
```


## Colors and backgrounds

```html
<!-- Text and background color helpers -->
<p class="text-primary">Primary text</p>
<p class="text-success">Success text</p>
<div class="p-2 bg-dark text-white rounded">Dark section</div>
```


## Buttons

```html
<!-- Variants, outline, sizes -->
<button class="btn btn-primary">Primary</button>
<button class="btn btn-outline-secondary">Outline</button>
<button class="btn btn-success btn-sm">Small</button>
<button class="btn btn-danger btn-lg">Large</button>
```


## Forms

```html
<!-- Form controls, checks, input groups -->
<input type="text" class="form-control mb-2" placeholder="Text input">

<div class="form-check mb-2">
  <input class="form-check-input" type="checkbox" id="chk1">
  <label class="form-check-label" for="chk1">Check me</label>
</div>

<div class="input-group">
  <span class="input-group-text">@</span>
  <input type="text" class="form-control" placeholder="username">
  <button class="btn btn-outline-secondary" type="button">Go</button>
</div>
```


## Navbar and navs

```html
<!-- Navbar -->
<nav class="navbar navbar-expand-lg bg-body-tertiary px-3 rounded">
  <a class="navbar-brand" href="#">Brand</a>
  <button class="navbar-toggler" data-bs-toggle="collapse" data-bs-target="#navDemo">
    <span class="navbar-toggler-icon"></span>
  </button>
  <div id="navDemo" class="collapse navbar-collapse">
    <ul class="navbar-nav me-auto">
      <li class="nav-item"><a class="nav-link active" href="#">Home</a></li>
      <li class="nav-item dropdown">
        <a class="nav-link dropdown-toggle" data-bs-toggle="dropdown" href="#">Menu</a>
        <ul class="dropdown-menu">
          <li><a class="dropdown-item" href="#">Action</a></li>
        </ul>
      </li>
    </ul>
    <form class="d-flex"><input class="form-control me-2"><button class="btn btn-outline-success">Search</button></form>
  </div>
</nav>
```

```html
<!-- Tabs -->
<ul class="nav nav-tabs">
  <li class="nav-item"><button class="nav-link active" data-bs-toggle="tab" data-bs-target="#home">Home</button></li>
  <li class="nav-item"><button class="nav-link" data-bs-toggle="tab" data-bs-target="#profile">Profile</button></li>
</ul>
<div class="tab-content p-3 border border-top-0">
  <div id="home" class="tab-pane fade show active">Home content</div>
  <div id="profile" class="tab-pane fade">Profile content</div>
</div>
```


## Cards

```html
<div class="card" style="width: 18rem;">
  <img src="https://picsum.photos/288/160" class="card-img-top" alt="">
  <div class="card-body">
    <h5 class="card-title">Card title</h5>
    <p class="card-text">Quick example text.</p>
    <a href="#" class="btn btn-primary">Go</a>
  </div>
</div>
```


## Alerts and badges

```html
<div class="alert alert-warning alert-dismissible fade show" role="alert">
  <strong>Heads up!</strong> Warning alert.
  <button type="button" class="btn-close" data-bs-dismiss="alert"></button>
</div>

<h4>Inbox <span class="badge bg-secondary">4</span></h4>
<button class="btn btn-primary">
  Notifications <span class="badge bg-light text-dark">3</span>
</button>
```


## Dropdowns

```html
<div class="dropdown">
  <button class="btn btn-secondary dropdown-toggle" data-bs-toggle="dropdown">Dropdown</button>
  <ul class="dropdown-menu">
    <li><a class="dropdown-item" href="#">Action</a></li>
    <li><a class="dropdown-item" href="#">Another</a></li>
    <li><hr class="dropdown-divider"></li>
    <li><a class="dropdown-item" href="#">Something else</a></li>
  </ul>
</div>
```


## Modal

```html
<button class="btn btn-primary" data-bs-toggle="modal" data-bs-target="#demoModal">Open modal</button>
<div id="demoModal" class="modal fade" tabindex="-1">
  <div class="modal-dialog"><div class="modal-content">
    <div class="modal-header">
      <h5 class="modal-title">Modal title</h5>
      <button class="btn-close" data-bs-dismiss="modal"></button>
    </div>
    <div class="modal-body">Hello from the modal!</div>
    <div class="modal-footer">
      <button class="btn btn-secondary" data-bs-dismiss="modal">Close</button>
      <button class="btn btn-primary">Save changes</button>
    </div>
  </div></div>
</div>
```


## Accordion / collapse

```html
<div class="accordion" id="acc">
  <div class="accordion-item">
    <h2 class="accordion-header">
      <button class="accordion-button" data-bs-toggle="collapse" data-bs-target="#c1">Item 1</button>
    </h2>
    <div id="c1" class="accordion-collapse collapse show" data-bs-parent="#acc">
      <div class="accordion-body">Content 1</div>
    </div>
  </div>
</div>
```


## List group

```html
<ul class="list-group">
  <li class="list-group-item active" aria-current="true">Active</li>
  <li class="list-group-item">Item</li>
  <li class="list-group-item disabled">Disabled</li>
</ul>
```


## Breadcrumb and pagination

```html
<nav aria-label="breadcrumb">
  <ol class="breadcrumb">
    <li class="breadcrumb-item"><a href="#">Home</a></li>
    <li class="breadcrumb-item"><a href="#">Library</a></li>
    <li class="breadcrumb-item active" aria-current="page">Data</li>
  </ol>
</nav>

<nav aria-label="pagination">
  <ul class="pagination">
    <li class="page-item disabled"><span class="page-link">Previous</span></li>
    <li class="page-item"><a class="page-link" href="#">1</a></li>
    <li class="page-item active"><span class="page-link">2</span></li>
    <li class="page-item"><a class="page-link" href="#">3</a></li>
    <li class="page-item"><a class="page-link" href="#">Next</a></li>
  </ul>
</nav>
```


## Toasts

```html
<div class="position-fixed bottom-0 end-0 p-3" style="z-index: 1080;">
  <div class="toast" role="alert" aria-live="assertive" aria-atomic="true">
    <div class="toast-header">
      <strong class="me-auto">Demo</strong>
      <small>now</small>
      <button class="btn-close ms-2 mb-1" data-bs-dismiss="toast"></button>
    </div>
    <div class="toast-body">Hello, world! Toast message.</div>
  </div>
</div>
```


## Progress and spinners

```html
<div class="progress my-3">
  <div class="progress-bar progress-bar-striped progress-bar-animated" style="width: 60%;">60%</div>
</div>

<div class="spinner-border text-primary" role="status" aria-label="loading"></div>
<div class="spinner-grow text-success" role="status" aria-label="loading"></div>
```


## Utilities you’ll use a lot

```html
<!-- Borders and radius -->
<div class="border rounded p-3 mb-2">Border + rounded</div>

<!-- Shadows -->
<div class="shadow-sm p-3 mb-2 bg-body rounded">Small shadow</div>

<!-- Width/height -->
<div class="w-50 h-25 bg-light">w-50 h-25</div>

<!-- Text alignment and transform -->
<p class="text-center text-uppercase">Centered uppercase</p>

<!-- Positioning -->
<div class="position-relative">
  <button class="position-absolute top-0 end-0 btn btn-sm btn-outline-secondary">X</button>
</div>
```

---

# Links
![[Lessons/3 - Front-end/Day 21/__blocks/Links]]