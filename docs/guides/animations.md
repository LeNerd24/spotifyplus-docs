---
sidebar_position: 6
sidebar_label: 'Animations and gestures'
---

# Animations and gestures

If you want your UI to move smoothly, you do not have to send a React update for every frame. Spotify Plus has shared values, animated styles, and gestures that run through its native animation runtime.

For new extensions, use `spotifyplus/react/reanimated`. `spotifyplus/react/Animated` is an alias for the same API. `spotifyplus/react/Gesture` provides the gesture builders. The lowercase `spotifyplus/react/animated` module is the older `Animated.Value` API; its values are a different type.

## Animate a view

```tsx
import React from 'react';
import { Button, View } from 'spotifyplus/react';
import Animated, { useSharedValue, useAnimatedStyle, withTiming } from 'spotifyplus/react/reanimated';

export function AnimatedCard() {
    const opacity = useSharedValue(1);
    const style = useAnimatedStyle(() => ({ opacity: opacity.value }));

    return (
        <View style={{ padding: 16 }}>
            <Animated.View style={[{ height: 80, backgroundColor: '#1ed760' }, style]} />
            <Button text="Fade" onPress={() => {
                opacity.value = withTiming(0.3, { duration: 300 });
            }} />
        </View>
    );
}
```

`useSharedValue()` keeps a value outside React state. `useAnimatedStyle()` reads it to update the native view. Assigning `withTiming()` starts an animation; `withSpring()` uses a spring instead. Use `cancelAnimation(value)` to stop an animation on that shared value.

The current build tools transform supported animation callbacks into worklets. A worklet runs in a separate runtime, so it cannot freely access your extension's Node APIs or arbitrary JavaScript objects. Keep captured data small and serializable. Use `runOnJS(callback)` when a worklet needs to call back into your extension. Build API 2 extensions through `spotifyplus build` or the generated scripts; copying an untransformed bundle is not enough.

## Add a gesture

```tsx
import React from 'react';
import Animated, { GestureDetector, useAnimatedStyle, useSharedValue } from 'spotifyplus/react/reanimated';
import { Gesture } from 'spotifyplus/react/Gesture';

export function DraggableCard() {
    const x = useSharedValue(0);
    const pan = Gesture.Pan().onUpdate(event => {
        x.value = event.translationX ?? 0;
    });
    const style = useAnimatedStyle(() => ({ transform: [{ translateX: x.value }] }));

    return (
        <GestureDetector gesture={pan}>
            <Animated.View style={[{ width: 80, height: 80, backgroundColor: '#1ed760' }, style]} />
        </GestureDetector>
    );
}
```

`GestureDetector` takes one React element. Gesture callbacks are worklets too. Use the runtime-bound detector exported from `reanimated` inside Spotify Plus.

## More animation APIs

The [animation reference](../spotifyplus-api/react/animations.md) covers shared values, hooks, timing, springs, interpolation, layout animations, runtime scheduling, and native view helpers. See [gestures](../spotifyplus-api/react/gestures.md) for gesture builders and callbacks, and [React components](../spotifyplus-api/react/components.md) for the available views.

These APIs run in Spotify Plus's renderer, not React Native. Android-only features require the native host. An exported name does not guarantee that another platform or a hostless JavaScript adapter implements it; unsupported operations throw `UnsupportedPlatformError`. Test animations and gestures on the Spotify build you support.
