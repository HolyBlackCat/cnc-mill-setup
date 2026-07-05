# gSender Macros

## Probe left edge X-2
```
(store current G90/G91 mode)
%ABSREL=modal.distance
(switch to relative distances)
G91

(probe)
(even if this fails, gSender somehow restores the original G90/G91 state)
G38.2 X10 F50
(offset is 1, plus 1 probe radius, resulting in final offset of 2 from the measurement point)
G0 X-1

(restore the original G90/G91 mode, interestingly this is untyped)
[ABSREL]
```

## Probe right edge X+2
```
(store current G90/G91 mode)
%ABSREL=modal.distance
(switch to relative distances)
G91

(probe)
(even if this fails, gSender somehow restores the original G90/G91 state)
G38.2 X-10 F50
(offset is 1, plus 1 probe radius, resulting in final offset of 2 from the measurement point)
G0 X1

(restore the original G90/G91 mode, interestingly this is untyped)
[ABSREL]
```

## Probe front edge Y-2
```
(store current G90/G91 mode)
%ABSREL=modal.distance
(switch to relative distances)
G91

(probe)
(even if this fails, gSender somehow restores the original G90/G91 state)
G38.2 Y10 F50
(offset is 1, plus 1 probe radius, resulting in final offset of 2 from the measurement point)
G0 Y-1

(restore the original G90/G91 mode, interestingly this is untyped)
[ABSREL]
```

## Probe back edge Y+2
```
(store current G90/G91 mode)
%ABSREL=modal.distance
(switch to relative distances)
G91

(probe)
(even if this fails, gSender somehow restores the original G90/G91 state)
G38.2 Y-10 F50
(offset is 1, plus 1 probe radius, resulting in final offset of 2 from the measurement point)
G0 Y1

(restore the original G90/G91 mode, interestingly this is untyped)
[ABSREL]
```

## X/2
```
G10 L20 X[posx/2]
```

## Y/2
```
G10 L20 Y[posy/2]
```

## Travel to max Z
```
(Using the jog command to be able to specify feed rate without changing the parser state.)
$J=G53 Z0 F1000
```
