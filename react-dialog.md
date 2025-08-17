# Dialog Component (React)

This component provides a simple modal dialog implementation using **React Portals**.  
It ensures that when the dialog is open, the background content does not scroll.

## Code

```jsx
import { useEffect } from "react"
import { createPortal } from "react-dom"

const Dialog = ({ children, isOpen }) => {
    useEffect(() => {
        if (isOpen) {
            document.body.style.overflowY = "hidden"
        } else {
            document.body.style.overflowY = "unset"
        }
        return () => {
            document.body.style.overflowY = "unset"
        }
    }, [isOpen])

    if (!isOpen) return null

    return (
        createPortal(
            <aside 
                className="fixed inset-0 w-full h-svh z-[5] bg-black/25 overflow-y-auto overflow-x-hidden flex items-center justify-center"
                role="dialog"
            >
                {children}
            </aside>,
            document?.body
        )
    )
}

export default Dialog
````

## Features

* Uses **React Portals** to render the dialog outside of the normal DOM hierarchy.
* Prevents background scrolling when the dialog is open (`overflowY: hidden`).
* Restores scrolling when the dialog closes.
* Accessible with `role="dialog"`.
* Full-screen overlay with dark translucent background.
* Children are **centered with flexbox**.

## Usage

```jsx
import { useState } from "react"
import Dialog from "./Dialog"

function App() {
    const [isOpen, setIsOpen] = useState(false)

    return (
        <div>
            <button onClick={() => setIsOpen(true)}>Open Dialog</button>

            <Dialog isOpen={isOpen}>
                <div className="bg-white p-6 rounded-lg max-w-md w-full">
                    <h2 className="text-xl font-bold">Hello!</h2>
                    <p>This is a dialog box.</p>
                    <button onClick={() => setIsOpen(false)}>Close</button>
                </div>
            </Dialog>
        </div>
    )
}
```

## Notes

* Tailwind classes (`flex`, `items-center`, `justify-center`) ensure dialog content is perfectly centered.
* Replace or extend styles as needed for your project.
* If multiple dialogs are needed, consider handling **focus trapping** for accessibility.
