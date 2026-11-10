---
sidebar_label: 'React components'
---

# React components

Spotify Plus provides a collection of React components for building native Android interfaces. These components work similarly to React Native, but render directly inside Spotify.

All components can be imported from `spotifyplus/react`.

```tsx
import { View, Text, Image, Pressable } from 'spotifyplus/react';
```

Unlike React for the web, Spotify Plus does not use HTML elements such as `<div>`, `<p>`, or `<img>`. Instead, use the native components provided by Spotify Plus.

If you're new to building interfaces with Spotify Plus, start with the [React UI guide](../../guides/ui.md).

## Layout

Layout components control how elements are positioned and organized.

| Component | Description |
| --- | --- |
| `View` | General-purpose container for other components. |
| `Row` | Arranges children horizontally. |
| `Column` | Arranges children vertically. |
| `ScrollView` | A vertically scrollable container. |
| `HorizontalScrollView` | A horizontally scrollable container. |
| `SafeAreaView` | A container that accounts for system insets. |
| `Space` | Adds empty space between elements. |

Additional layout components include `PlainView`, `FrameLayout`, `RelativeLayout`, `LinearLayout`, `HorizontalStackLayout`, and `VerticalStackLayout`. These provide more direct access to Android's native layout containers.

### Example

```tsx
import { View, Row, Column, Text, Image } from 'spotifyplus/react';

export function TrackInfo() {
    return (
        <Row style={{ padding: 16 }}>
            <Image source={{ uri: 'https://example.com/cover.png' }} style={{ width: 64, height: 64 }} />
            <Column style={{ marginLeft: 12 }}>
                <Text>Song Title</Text>
                <Text>Artist Name</Text>
            </Column>
        </Row>
    );
}
```

## Text and images

| Component | Description |
| --- | --- |
| `Text` | Displays text. |
| `TextView` | Native Android text view. |
| `Image` | Displays an image from a URI or extension asset. |
| `ImageView` | Native Android image view. |

Use `Text` and `Image` for most interfaces. The corresponding Android view components are also available when you need them.

## User input

Spotify Plus includes components for buttons, text input, and selection controls.

| Component | Description |
| --- | --- |
| `Button` | A button with a text label. |
| `ImageButton` | A button that displays an image. |
| `TextInput` | Editable text input. |
| `EditText` | Native Android text input. |
| `Switch` | On/off toggle. |
| `CheckBox` | Checkbox selection. |
| `RadioButton` | Selectable option within a group. |
| `RadioGroup` | Groups radio buttons together. |
| `ToggleButton` | Button with a checked state. |
| `Slider` | Selects a numeric value by dragging. |
| `SeekBar` | Native Android slider. |

### Pressable

`Pressable` is a flexible component for handling touch interactions. Unlike a standard button, it can wrap other components and customize its appearance based on whether it is pressed or focused.

```tsx
import { Pressable, Text } from 'spotifyplus/react';

export function CustomButton() {
    return (
        <Pressable onPress={() => console.log('Pressed!')}>
            <Text>Click me</Text>
        </Pressable>
    );
}
```

Supported events include `onPress`, `onLongPress`, `onPressIn`, and `onPressOut`.

Both `style` and `children` can also receive a function with the current `{ pressed, focused }` state.

`TouchableOpacity` provides similar functionality, with an additional `activeOpacity` property to control the opacity when pressed.

## Lists and scrolling

For simple scrolling content, use `ScrollView` or `HorizontalScrollView`.

For larger collections of items, use `FlatList`. Unlike a regular scroll view, `FlatList` renders items in a window around the visible area, making it better suited for long lists.

```tsx
import { FlatList, Text } from 'spotifyplus/react';

const songs = [
    { id: '1', title: 'First Song' },
    { id: '2', title: 'Second Song' },
    { id: '3', title: 'Third Song' }
];

export function SongList() {
    return (
        <FlatList
            data={songs}
            keyExtractor={item => item.id}
            renderItem={({ item }) => <Text>{item.title}</Text>}
        />
    );
}
```

`FlatList` also supports headers, footers, empty states, separators, and programmatic scrolling.

For more control over rendering performance, see the exported `FlatList` prop types.

## Progress indicators

| Component | Description |
| --- | --- |
| `ActivityIndicator` | Displays an indeterminate loading indicator. |
| `ProgressBar` | Native Android progress indicator. |
| `ProgressBarHorizontal` | Horizontal progress bar. |

Use these components to display loading states or the progress of an ongoing operation.

## Bottom sheets

`BottomSheet` displays content in a panel that slides up from the bottom of the screen.

```tsx
import { useState } from 'react';
import { BottomSheet, Button, Text, View } from 'spotifyplus/react';

export function Options() {
    const [visible, setVisible] = useState(false);

    return (
        <View>
            <Button text="Open options" onPress={() => setVisible(true)} />
            <BottomSheet visible={visible} onDismiss={() => setVisible(false)}>
                <View style={{ padding: 24 }}>
                    <Text>My options</Text>
                </View>
            </BottomSheet>
        </View>
    );
}
```

The sheet's visibility is controlled through the `visible` prop. You can also configure whether it can be dragged or dismissed by tapping outside.

A ref to the component provides `show()` and `hide()` methods for controlling it programmatically.

## Color picker

`ColorPicker` provides a native color selection interface with saturation and hue controls.

```tsx
import { ColorPicker } from 'spotifyplus/react';

export function AccentPicker() {
    return (
        <ColorPicker
            color="#c7a1ef"
            style={{ height: 240 }}
            onColorChangeComplete={event => {
                SpotifyPlus.Theme.set({ colors: { accent: event.color } });
            }}
        />
    );
}
```

Two events are available:

- `onColorChange` — Fires continuously as the user adjusts the color.
- `onColorChangeComplete` — Fires when the user finishes adjusting the color.

Both events provide the selected color as a hexadecimal string, along with its hue, saturation, and value.

## Custom native components

Extensions with native Android code can expose their own React components using `createNativeComponent`.

```tsx
import { createNativeComponent } from 'spotifyplus/react';

const ArtworkGradient = createNativeComponent<{
    topColor: string;
    bottomColor: string;
}>('ArtworkGradient');
```

The component name must match a component registered by the native plugin.

You can then use it like any other React component:

```tsx
// <ArtworkGradient topColor="#402060" bottomColor="#101010" />
```

## Custom drawing

For interfaces that need custom graphics, Spotify Plus provides three drawing components:

| Component | Description |
| --- | --- |
| `ScriptView` | Renders custom graphics from a display list. |
| `RenderView` | Renders custom graphics from a display list. |
| `CanvasView` | Renders custom graphics from a display list. |

These components support shapes, paths, text, images, gradients, transforms, clipping, and shadows.

They also provide touch events, frame callbacks, and size change notifications.

## Styling

Components can be styled using the `style` prop.

Spotify Plus also exports `StyleSheet` for organizing and reusing styles.

```tsx
import { StyleSheet, View, Text } from 'spotifyplus/react';

const styles = StyleSheet.create({
    container: {
        padding: 16,
        backgroundColor: '#181818'
    },
    title: {
        color: '#ffffff',
        fontSize: 20
    }
});

export function Example() {
    return (
        <View style={styles.container}>
            <Text style={styles.title}>Hello, Spotify Plus!</Text>
        </View>
    );
}
```

Numeric layout dimensions use density-independent sizing, so you generally don't need to convert values to physical Android pixels.

`StyleSheet` also provides `flatten`, `absoluteFill`, and `absoluteFillObject` for working with styles.

For the full list of supported properties and events, refer to the TypeScript definitions exported by `spotifyplus/react`.