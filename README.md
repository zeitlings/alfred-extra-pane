<h1 align="center">
  
<a href="https://github.com/mr-pennyworth/alfred-extra-pane/releases/latest/">
  <img src="AlfredExtraPane/Assets.xcassets/AppIcon.appiconset/icon_512x512.png" width="16%"><br/>
  <img alt="Download"
       src="https://img.shields.io/github/downloads/mr-pennyworth/alfred-extra-pane/total?color=purple&label=Download"><br/>
</a>
  AlfredExtraPane
</h1>

Spotight-like rich previews for [Alfred](https://alfredapp.com) workflows.

![](media/demo-fast.gif)

`AlfredExtraPane` is an experimental app that renders HTML from
`quicklookurl` of every item in the JSON produced by
[Alfred Workflows](https://www.alfredapp.com/workflows/).

#### How does it do it?
Alfred has an experimental "[Press Secretary](https://www.alfredforum.com/topic/16111-wip-poc-spotlight-like-rich-preview-pane-for-alfred-workflows/?do=findComment&comment=83222)" to publish
macOS [distributed notifications](https://developer.apple.com/documentation/foundation/distributednotificationcenter). These notifications
contain all the information needed to be able to show the extra pane.

#### Adding to workflow
Any workflow that produces items with `quicklookurl`s that are either
HTML files or HTTP links automatically causes the extra pane to show
up with the `quicklookurl` loaded in it.

#### Alfred theme support
Alfred's themes are stored in JSON files. Here's a snippet from one such file:
```json
{
  "alfredtheme" : {
    "result" : {
      "textSpacing" : 10,
      "subtext" : {
        "size" : 11,
        "colorSelected" : "#6E7073FF",
        "font" : "System Light",
        "color" : "#6E7073E5"}}}}
```

The pane converts this into CSS variables and injects them into the HTML.
The CSS looks like this:
```css
:root {
  --result-textSpacing: 10px;
  --result-subtext-size: 11px;
  --result-subtext-colorSelected: "#6E7073FF";
  --result-subtext-font: "System Light";
  --result-subtext-color: "#6E7073E5";
}
```
As a workflow author, when you generate the HTML, use these
variables in it. The pane will make sure they're injected.
Check out the [tutorial](#tutorial) for a quick walk-through.

## What's not yet supported?
 - Configurability
    - Workflows should be able to opt out of the extra pane

## Configuration
The global pane(s) can be configured by editing
`{/path/to}/Alfred.alfredpreferences/preferences/mr.pennyworth.AlfredExtraPane/config.json`.
Similarly, pane(s) for individual workflows can be configured by editing
`{workflow-dir}/extra-pane-config.json`. If there are any workflow-specific
panes, the items produced from that workflow will not be shown in the global
panes.

Alternatively, you can use the `Configure > Global` menu to open the global
config JSON file in your default editor. Similarly,
`Configure > [Workflow Name]` opens the workflow-specific config file.
![](media/configure-menu.png)

Configurable parameters are:
 - `alignment` (required):
    - `horizontal`
       - `placement`: `left` or `right`
       - `width`: width of the pane
       - `minHeight`: minimum height of the pane
    - `vertical`
       - `placement`: `top` or `bottom`
       - `height`: height of the pane
 - `customUserAgent` (optional): User-Agent string for HTTP(S) URLs

Here's an example with four panes configured:
```json
[{
  "alignment" : {
    "horizontal" : {"placement" : "right", "width" : 300, "minHeight" : 400}}
}, {
  "alignment" : {
    "horizontal" : {"placement" : "left", "width" : 300, "minHeight" : 400}}
}, {
  "alignment" : {
    "vertical" : {"placement" : "top", "height" : 100}}
}, {
  "alignment" : {
    "vertical" : {"placement" : "bottom", "height" : 200}}
}]
```
![](media/multi-pane.png)

## Tutorial
Here's a script filter that produces a result:
![](media/tutorial-images/script.png)

This is what you get when you run it: ![](media/tutorial-images/alfred-1.png)

Now, let's attch an HTML preview to this result.
Create `/tmp/one.html`:
```html
<html>
  <head>
  </head>
  <body>
    <h1> One </h1>
  </body>
</html>
```
Change the script filter:
```bash
cat << EOF
{"items" : [
  {"title": "One",
   "quicklookurl": "/tmp/one.html"}
]}
EOF
```
And the preview shows up!
![](media/tutorial-images/alfred-3.png)

Now let's make the preview blend-in with the theme.
Here's a snippet from relevant parts of Alfred's theme:
```json
{
  "alfredtheme" : {
    "result" : {
      "backgroundSelected" : "#00000054",
      "text" : {
        "size" : 22,
        "colorSelected" : "#E1E1E2FF",
        "font" : "System Light",
        "color" : "#A8A8ABFF"}}}}
```

Looking at the variable names in the above JSON, add the style section
to `/tmp/one.html`:
```html
<html>
  <head>
    <style>
      h1 {
          color: var(--result-text-colorSelected);
      }
    </style>
  </head>
  <body>
    <h1> One </h1>
  </body>
</html>
```

Themed preview should show up:
![](media/tutorial-images/alfred-4.png)
