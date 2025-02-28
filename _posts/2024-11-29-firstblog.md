---
layout: post
title: "Elegant Messagebox Made Simple with React Hooks and Context API (Nextjs)"
date: 2024-11-29
---

I've always wanted an easy to use message box component for my react projects. However it is not always easy to find a simple and elegant solution.

I have tried many libraries and components but none of them were exactly what I wanted. So I decided to create my own. In this article I will show you how to create a simple and elegant message box component using React Hooks and Context API.

### Step 1: A Promise, deferred

It is very important that this message box can be used like a promise, meaning it should be as simple as:
```typescript
const confirmed = await askToConfirm('Are you sure?');
```

At the start I thought maybe a simple promise is enough, but soon I realized that I need to be able to resolve the promise elsewhere in the code, the solution is then obvious, a deferred object.

```typescript
class Deferred<T> {
    resolve: (value: T) => void = () => {};
    reject: (reason?: any) => void = () => {};
    promise: Promise<T>;

    constructor() {
        this.promise = new Promise((resolve, reject)=> {
            this.reject = reject
            this.resolve = resolve
        })
    }
}
```

Then you can use the deferred object like this:
```typescript
const deferred = new Deferred<boolean>();

// Some where else in the code
deferred.resolve(true);
// or
deferred.reject();
```

### Step 2: The Messagebox

This is the simple part, you create a message box component that suits your own taste. Here is mine using shadcn:
```tsx
import {
    Dialog,
    DialogContent,
    DialogHeader,
    DialogTitle,
    DialogFooter,
} from "@/components/ui/dialog";

interface YesNoBoxProps {
    title: string;
    message: string;
    open: boolean;
    handleConfirm: () => void;
    handleCancel: () => void;
}
const YesNoBox = ({
    title,
    message,
    open,
    handleCancel,
    handleConfirm
}: YesNoBoxProps) => {

    return (
        <Dialog open={open} onOpenChange={handleCancel}>
            <DialogContent>
                <DialogHeader>
                    <DialogTitle>
                        {title}
                    </DialogTitle>
                </DialogHeader>
                {message}

                <DialogFooter>
                    <Button
                        onClick={handleCancel}
                        variant={'outline'}>Cancel</Button>
                    <Button
                        onClick={handleConfirm}
                    >Confirm</Button>
                </DialogFooter>
            </DialogContent>
        </Dialog>
    );
}
```

### Step 3: The Context

What I wanted is that the messagebox component is mounted at the root layout, and can be triggered from anywhere in the code. This is where the context API comes in handy.

```typescript
interface MessageBoxContextType {
    askToConfirm: (title: string, message: string) => Promise<boolean>;
}
const MessageBoxContext = React.createContext<MessageBoxContextType | undefined>(undefined);
```

### Step 4: The Provider

The provider should handle the state of the message box, and also the deferred object.

```tsx
export const MessageBoxProvider = ({children}: {children: React.ReactNode}) => {
    const [open, setOpen] = useState(false);
    const [title, setTitle] = useState('');
    const [message, setMessage] = useState('');
    const [deferred, setDeferred] = useState(new Deferred());

    const handleCancel = () => {
        deferred.resolve(false);
        setOpen(false);
        setDeferred(new Deferred());
    };

    const handleConfirm = () => {
        deferred.resolve(true);
        setOpen(false);
        setDeferred(new Deferred());
    };

    const askToConfirm = (title: string, message: string) => {
        setTitle(title);
        setMessage(message);
        setOpen(true);
        return deferred.promise;
    };

    {% raw %}
    return (
        <MessageBoxContext.Provider value={{ askToConfirm }}>
            {children}
            <YesNoBox
                title={title}
                message={message}
                open={open}
                handleCancel={handleCancel}
                handleConfirm={handleConfirm}
            />
        </MessageBoxContext.Provider>
    );
    {% endraw %}
}
```

In the provider, the deferred object is set to the context's state, the handleCancel/Confirm functions resolve the deferred object and reset the state.

### Step 5: The Hook

The hook is simple, it just returns the askToConfirm function from the context.

```typescript
export const useYesNoBox = () => {
    const context = useContext(MessageBoxContext);
    if (!context) {
        throw new Error('useYesNoBox must be used within a MessageBoxProvider');
    }
    return context;
}
```

### Step 6: Usage

First you need to put the provider in a layout, for instance `app/layout.tsx`:
```tsx
import type { Metadata } from 'next'
import { Inter } from 'next/font/google'
import './globals.css'
import {MessageBoxProvider} from '@/components/MessageBox';

const inter = Inter({ subsets: ['latin'] })

export const metadata: Metadata = {
    title: 'Simple MessageBox',
    description: 'Generated by create next app',
}

export default function RootLayout({
    children,
}: { children: React.ReactNode }) {

    return (
        <html lang="en">
            <body className={inter.className}>
                <MessageBoxProvider>
                    <main className='flex w-screen justify-center'>
                        {children}
                    </main>
                </MessageBoxProvider>
            </body>
        </html>
    )
}
```

Then you can use the hook in any component:
```tsx
import {useYesNoBox} from '@/components/MessageBox';

export default function Home() {
    const {askToConfirm} = useYesNoBox();

    const handleConfirm = async () => {
        const confirmed = await askToConfirm('Are you sure?', 'This action cannot be undone');
        if (confirmed) {
            console.log('Confirmed');
        } else {
            console.log('Cancelled');
        }
    }

    return (
        <div>
            <button onClick={handleConfirm}>Confirm</button>
        </div>
    )
}
```
Simple and easy to use, just the way I like it.


### The full code
```tsx
'use client';
import React, { createContext, useContext, useState, ReactNode } from "react";
import {
    Dialog,
    DialogContent,
    DialogHeader,
    DialogTitle,
    DialogFooter,
} from "@/components/ui/dialog";
import {Button} from "@/components/ui/button";

export class Deferred<T> {
    resolve: (value: T) => void = () => {};
    reject: (reason?: any) => void = () => {};
    promise: Promise<T>;

    constructor() {
        this.promise = new Promise((resolve, reject)=> {
            this.reject = reject
            this.resolve = resolve
        })
    }
}

type YesNoBoxProps = {
    title: string;
    message: string;
    open: boolean;
    handleConfirm: () => void;
    handleCancel: () => void;
}
const YesNoBox = ({
    title,
    message,
    open,
    handleCancel,
    handleConfirm
}: YesNoBoxProps) => {

    return (
        <Dialog open={open} onOpenChange={handleCancel}>
            <DialogContent>
                <DialogHeader>
                    <DialogTitle>
                        {title}
                    </DialogTitle>
                </DialogHeader>
                {message}

                <DialogFooter>
                    <Button
                        onClick={handleCancel}
                        variant={'outline'}>Cancel</Button>
                    <Button
                        onClick={handleConfirm}
                    >Confirm</Button>
                </DialogFooter>
            </DialogContent>
        </Dialog>
    )
}

interface DialogContextProps {
    askToConfirm: (title: string, message: string) => Promise<boolean>;
}
const DialogContext = createContext<DialogContextProps | undefined>(undefined);

export const MessageBoxProvider = ({children}: {children: React.ReactNode}) => {
    const [open, setOpen] = useState(false);
    const [title, setTitle] = useState('');
    const [message, setMessage] = useState('');
    const [deferred, setDeferred] = useState(new Deferred<boolean>());

    const handleCancel = () => {
        deferred.resolve(false);
        setOpen(false);
        setDeferred(new Deferred());
    };

    const handleConfirm = () => {
        deferred.resolve(true);
        setOpen(false);
        setDeferred(new Deferred());
    };

    const askToConfirm = (title: string, message: string) => {
        setTitle(title);
        setMessage(message);
        setOpen(true);
        return deferred.promise;
    };

    {% raw %}
    return (
        <DialogContext.Provider value={{ askToConfirm }}>
            {children}
            <YesNoBox
                title={title}
                message={message}
                open={open}
                handleCancel={handleCancel}
                handleConfirm={handleConfirm}
            />
        </DialogContext.Provider>
    );
    {% endraw %}
}

export const useYesNoDialog = () => {
    const context = useContext(DialogContext);
    if (!context) {
        throw new Error('useYesNoDialog must be used within MessageBoxProvider');
    }
    return context;
}
```

Happy coding! I hope you find this article helpful.
