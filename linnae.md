---
title: LinnaeUI
---

A minimal, composable application UI framework.

# Fundamental particles

## Containers

Manage the display of child elements, handling sizing, positioning and scroll.

- **Grid**s divide space horizontally, vertically or both, and place each child in its own exclusive space. This includes one-dimensional “grids” (e.g. flex boxes). See [Fluid Layout](/fluid).
- **Stack**s render all their children “stacked on top of one another” in the same space. The order of elements may be dynamically switched to bring different children to the top. Multiple children may be visible at a time if they have transparent regions. Children have absolute positions in the stack.

## Content

Visible elements, handling their internal sizing.

- **Text**s re-flow when resized, and can be nested to create formatted spans.
- **Graphic**s may be bitmap or vector images, videos or animations. May have complex sizing behaviors, especially for vector images.

## Interactions

Interaction elements take one child, and gives it user interaction capability.

- **Pressable**, e.g. a button or menu item. Triggers a single event on tap, click, Enter or Space keys. Also includes secondary taps using right click, long press or modifier key during click/Enter.
- **Editable**, e.g. typing into a text input. The element contains a text value which is updated based on keyboard input while focused. Clicks and taps move the caret, while drags select text.
- **Scrollable**, i.e. move an element in space using the mouse wheel or keys. Scrolls automatically when required to bring focused elements into view.
- **Draggable**, i.e. move an element in space using pointer drag or directional keys. This includes controls like sliders and sizing handles.

All interaction elements support **focus**, which makes it sensitive to keyboard input. A click or tap focuses an element, and focus may be moved between neighbors using Tab or other keys. All behavior elements also support **hover** so that child elements may provide visual feedback of cursor position.

```py
# Here is a datepicker component in LinnaeUI

def datepicker():
  stack
    boxgraphic
    editable(focused -> dropdownShown)
      text YYYY-MM-DD
    dropdown(shown <- dropdownShown)
      grid
        ...

# Here we are unidirectionally binding the
# "focused" state property of the editable
# to the "shown" property of the dropdown. 
```

---

# Atoms

- **Text**: Shows text 🤷
- **Icon**: Shows an icon. Props `name` and `size`. Icons must be SVGs and render using currentColor.
- **Number**: Uses Intl.NumberFormat, and can display currencies, percentages, units or just plain numbers. Props include `value` (a number) and all Intl.NumberFormat options
- **Date**: Uses Intl.DateTimeFormat or Intl.RelativeDateFormat; Props include `value` (a Date, numeric timestamp or Temporal), a `relative` flag and Intl options
- **Button**, which responds to clicks and not much else. Props include `active` (boolean), `appearance` (normal, primary, basic), DOM attributes and children.
- **Input**, which allows users to edit text within them. These are rendered without borders. Props include the `multiline` flag, which switches to a textarea, and DOM attributes.
- **Popover**: Displays children attached to an *anchor element.*
- **Grid**: Displays children in a CSS grid. Accepts the grid template as a prop.

# Molecules

- **Calendar**, which uses a Grid, Numbers and Intl APIs.
- **Table**, which uses a grid to show children. Supports Grid props as well as integer props `fixTop`, `fixLeft`, `fixRight` and `fixBottom`
- **Field**, which shows some read-only data in the *closed* state, and switches to an *open* state for editing. Props include `label` (string), `message` (string), `validation` (none, error, warning, success), `open` (boolean) and up to two `children`. When open, the first child is shown in the input area and the second in the popover.
- **TextField** uses Field to provide text editing, with an overflowing, auto-sizing text area when open. Props include `value` and `onChange`.
- **NumberField**, with props like `value`, `onChange`, Field props and Date props
- **DateField**, which may render a calendar in the field popover. Props include `value`, `onChange`, Field props and Date props
- **SelectInput**, with Field props and `multi`, `searchable`, `choices`, `value`, `onChange` and `onSearch`

# Patterns

- **Navigation** or moving between data without changing it
    - **Show** new elements without removing existing ones (e.g. pop-up menu button)
    - **Hide** some element (e.g. close button)
    - **Swap** existing elements for new ones, where the navigation control itself remains visible. (e.g. tabs)
    - **Move** to a new location, replacing existing elements, including the navigation control itself, with new ones. (e.g. link)
- **Indicators**, which display information
- **Triggers** which start tasks (e.g. submit button)

