# Canvas

The `canvas` component provides a 2D drawing context, similar to the `<canvas>` element in HTML5. It allows 2D shapes and bitmap images to be rendered dynamically through scripts.

## Usage

```xml
<canvas id="myCanvas" width="300" height="150"></canvas>
```

In your JavaScript:

```javascript
// Get the canvas instance
const canvas = this.selectComponent('#myCanvas');
// Get the 2D drawing context
const ctx = canvas.getContext('2d');

ctx.fillStyle = 'red';
ctx.fillRect(10, 10, 150, 75);
```

## Properties

| Property | Type | Description | Default |
|-----------|------|-------------|---------|
| `width` | Number | The pixel width of the canvas. | `300` |
| `height` | Number | The pixel height of the canvas. | `150` |

## API

The `canvas` component is controlled through the standard Web Canvas API. You can use `canvas.getContext('2d')` to get the drawing context.

For a detailed API list, see the [Canvas API Specification](/AIUI/api/canvas).
