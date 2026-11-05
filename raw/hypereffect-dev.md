---
url: https://hypereffect.dev/
date_fetched: 2026-11-06
---

# hypereffect

          A simple protocol for describing server‑guided, reactive interfaces that can be
          **rendered anywhere**.
        

- 
            A client, the *renderer*, starts the session by sending**hello**. It knows nothing about the application yet.renderer → server`hello {}`
- 
            The server answers with the state, as signals, and a few **elements**. The renderer draws them with its own native widgets, in its own style.There is a **fixed set of eighteen components**, and they carry**semantic variants, not styles**:`h2`or`warning`, never a colour or a margin.server → renderer`signal {"moisture":24,"_why":false} element {"id":"plant","type":"column"} element {"id":"name","type":"text","parent":"plant","variant":"h2","text":"Basil"} element {"id":"soil","type":"notice","parent":"plant","variant":{"cel":"s.moisture < 30 ? 'warning' : 'success'"},"text":{"cel":"'Soil moisture ' + string(s.moisture) + '%'"}} element {"id":"water","type":"button","parent":"plant","variant":"primary","label":"Water now","onPress":{"action":"water"}}`
- 
            Elements declare up front the **few interactions they support**. When the user acts, a handler can do only two things:- 
                  **change the state locally**, and every attribute that depends on it is recomputed: pressing*Why?*sets`_why`, and the tip appears;
- 
                  **fire an action**, like`water`above.
 server → renderer`element {"id":"why","type":"button","parent":"plant","variant":"plain","label":"Why?","onPress":{"patch":{"_why":true}}} element {"id":"tip","type":"text","parent":"plant","variant":"caption","text":"Basil wilts below 30%.","visible":{"cel":"s._why"}}`
- 
                  
- 
            **Actions**are sent to the server, with the current signals. Local signals, whose names start with`_`, stay on the client.renderer → server`action {"name":"water","signals":{"moisture":24}}`
- 
            The server decides what happens next. It can send **new elements**,**replace**or**remove**existing ones, or send**signal updates**that are merged into the client's state, which may change the interface again.Here the pump has run. The notice now reads *Soil moisture 61%*and turns from a warning into a success without being re-sent: its text and variant are expressions.server → renderer`signal {"moisture":61} element {"id":"water","type":"button","parent":"plant","variant":"primary","label":"Watered","disabled":true} remove {"id":"why"}`
- 
            The server answers with the state, as signals, and a few **elements**. The renderer draws them with its own native widgets, in its own style.There is a **fixed set of eighteen components**, and they carry**semantic variants, not styles**:`h2`or`segmented`, never a colour or a margin.server → renderer`signal {"time":"20:00","notes":"","_notes":false} element {"id":"book","type":"column"} element {"id":"title","type":"text","parent":"book","variant":"h2","text":"Table for two"} element {"id":"time","type":"select","parent":"book","variant":"segmented","label":"Time","bind":"time","options":[{"value":"19:00","label":"19:00"},{"value":"20:00","label":"20:00"},{"value":"21:30","label":"21:30"}]} element {"id":"go","type":"button","parent":"book","variant":"primary","label":{"cel":"'Book for ' + s.time"},"onPress":{"action":"reserve"}}`
- 
            Elements declare up front the **few interactions they support**. When the user acts, a handler can do only two things:- 
                  **change the state locally**, and every attribute that depends on it is recomputed: picking*21:30*updates the bound`time`and the button now reads*Book for 21:30*; pressing*Add a note*sets`_notes`, and a notes field appears;
- 
                  **fire an action**, like`reserve`above.
 server → renderer`element {"id":"add","type":"button","parent":"book","variant":"plain","label":"Add a note","onPress":{"patch":{"_notes":true}}} element {"id":"notes","type":"input","parent":"book","variant":"multiline","label":"Notes","bind":"notes","visible":{"cel":"s._notes"}}`
- 
                  
- 
            **Actions**are sent to the server, with the current signals. Local signals, whose names start with`_`, stay on the client.renderer → server`action {"name":"reserve","signals":{"time":"21:30","notes":"Window seat, please"}}`
- 
            The server decides what happens next. It can send **new elements**,**replace**or**remove**existing ones, or send**signal updates**that are merged into the client's state, which may change the interface again.Here the table is booked. The new notice reads *Booked for 21:30, ref LP-4821*: its text is an expression over the time the user picked and the reference the server just sent.server → renderer`signal {"ref":"LP-4821"} element {"id":"done","type":"notice","parent":"book","before":"time","variant":"success","text":{"cel":"'Booked for ' + s.time + ', ref ' + s.ref"}} element {"id":"go","type":"button","parent":"book","variant":"primary","label":"Booked","disabled":true} remove {"id":"add"}`
- 
            The server answers with the state, as signals, and a few **elements**. The renderer draws them with its own native widgets, in its own style.There is a **fixed set of eighteen components**, and they carry**semantic variants, not styles**:`number`or`primary`, never a colour or a margin.server → renderer`signal {"amount":null,"kind":"meals"} element {"id":"exp","type":"column"} element {"id":"title","type":"text","parent":"exp","variant":"h2","text":"New expense"} element {"id":"amount","type":"input","parent":"exp","variant":"number","label":"Amount (€)","bind":"amount"} element {"id":"kind","type":"select","parent":"exp","label":"Category","bind":"kind","options":[{"value":"travel","label":"Travel"},{"value":"meals","label":"Meals"},{"value":"office","label":"Office supplies"}]} element {"id":"go","type":"button","parent":"exp","variant":"primary","label":"Submit","disabled":{"cel":"s.amount == null"},"onPress":{"action":"submit"}}`
- 
            Elements declare up front the **few interactions they support**. When the user acts, a handler can do only two things:- 
                  **change the state locally**, and every attribute that depends on it is recomputed: typing*64.50*for a meal shows the policy warning and enables*Submit*; pressing*Reset*patches`amount`back to empty, and both go away;
- 
                  **fire an action**, like`submit`above.
 server → renderer`element {"id":"policy","type":"notice","parent":"exp","before":"go","variant":"warning","text":"Meals over €50 need a manager's approval.","visible":{"cel":"s.kind == 'meals' && s.amount != null && s.amount > 50"}} element {"id":"reset","type":"button","parent":"exp","variant":"plain","label":"Reset","onPress":{"patch":{"amount":null}}}`
- 
                  
- 
            **Actions**are sent to the server, with the current signals. Local signals, whose names start with`_`, stay on the client.renderer → server`action {"name":"submit","signals":{"amount":64.5,"kind":"meals"}}`
- 
            The server decides what happens next. It can send **new elements**,**replace**or**remove**existing ones, or send**signal updates**that are merged into the client's state, which may change the interface again.Here the expense is filed and the server clears `amount`for the next one. The policy warning hides and*Submit*is disabled again on their own: both are expressions over`amount`.server → renderer`signal {"amount":null} element {"id":"sent","type":"notice","parent":"exp","before":"amount","variant":"success","text":"€64.50 for meals sent to your manager."}`

## One application, every renderer

Messages say what is on the screen, never how it looks. Each renderer draws them with the widgets and conventions of its own platform, so the same server looks at home in each of them.

          **Native look and feel** comes built in: you get each platform's own widgets,
          fonts, spacing and accessibility without any theming work.
        

Why

## Every program still builds its own interface

For decades, each new kind of computer came with an operating system written just for it, and the same work was repeated until a shared one took over. Phones went through it only recently, until Android.

Interfaces are still at that stage. Useful software that needs to take input from people and show them results invents its own interface, often from raw bytes on a screen or a serial line. Even with a cross-platform toolkit or a whole web browser underneath, the interface is still built from zero, once per program.

            **hypereffect is the shared layer.** It provides the bare minimum needed
            for most of what software asks of people, in a form that is safe and simple: a renderer
            is written once per platform and works with any number of servers.
          

## Transport-agnostic

hypereffect doesn't specify how the server talks to the renderer, or what a server and a renderer are in the first place. Anything that carries text reliably will do. The spec defines mappings for the common ones.

- **stdin / stdout**of a child process
- **Unix socket**for local services
- **Serial line**to an embedded device
- **SSE + HTTP POST**servers stream, renderers call
- **iroh**bidirectional streams
- **In-process**a WASM module

## Use cases

Interfaces that are mostly text, inputs, buttons and lists, with the logic on a server or in another process.

### Sandboxed modules

A WebAssembly module can render structured information and get input from a user without having to paint each pixel.

### Control panels for hardware

A device serves its own control panel over a socket or serial line, the controller gets a clear and native UI.

### Alternative browser ecosystem for simple services

A service speaks hypereffect at a URL and looks native in a generic hypereffect browser.

### Interfaces generated by agents

An LLM agent builds forms and choices on the fly, as plain JSON with no code to run.

## Non-use cases

- 
            **Standalone apps**that don't talk to any servers or third-party code.
- 
            **Visual, highly interactive interfaces.**Data visualization, editing photos, audio or video, action games, navigating a map: anything drawn or manipulated directly rather than described as text, inputs and buttons.
- 
            **Tools people live in.**Software used all day, where comfort, visual polish and shortcuts that build into muscle memory matter more than anything, even when the content is plain text.
- 
            **Dense, busy screens.**Applications that show a lot at once with many potential actions always within reach of the user, like dashboards, messaging and social media apps, trading screens, project boards.

## A hypermedia system

hypereffect strives to be the world's second most successful hypermedia system, after HTML.

- 
            ### The server describes the UI.The client receives the interface itself: it never parses contacts or orders and never needs to know what they mean in order to render them. 
- 
            ### The server drives application state.The server decides what the user sees and which actions are available next, instead of client code deciding what is possible. 
- 
            ### Behavior is declared in the markup.Each element describes the interactions it offers, the document is self-describing. Buttons and inputs are *hypermedia controls*.

## Deliberately small

The protocol leaves things out on purpose, so that every renderer can show every application.

- 
            ### No styling.Variants say what something is, like `h2`,`primary`or`warning`. The renderer decides how they look.
- 
            ### No custom components.18 components, the same in every renderer, interoperability reigns. 
- 
            ### No scripts.Expressions compute values and have no side effects. A server cannot run arbitrary code on the renderer. 

## Inherently safe

A renderer trusts a server for content, never for execution. There is nothing to run: expressions have no side effects and there are no scripts, so opening an unknown server is about as risky as reading what it says. In the other direction, the server treats every action as untrusted user input, whatever the renderer claims to have shown. See the security considerations.

## Necessarily accessible

Messages carry meaning, not pixels: headings have a level, buttons and inputs have labels, images have alt text. A renderer maps them to the platform's own widgets, so screen readers, keyboard navigation, text scaling and high contrast work the way people already know.

- `text · h2 · "Basil"`- Basil, heading level 2 
- 
            `notice · warning · "Soil moisture 24%"`Warning: Soil moisture 24% 
- 
            `button · primary · "Water now"`Water now, button 
- 
            `text · h2 · "Table for two"`Table for two, heading level 2 
- 
            `select · segmented · "Time"`Time, 21:30, 3 of 3 
- 
            `button · primary · "Book for 21:30"`Book for 21:30, button 
- 
            `text · h2 · "New expense"`New expense, heading level 2 
- 
            `input · number · "Amount (€)"`Amount (euros), edit text, 64.50 
- 
            `notice · warning · "Meals over €50…"`Warning: Meals over €50 need a manager's approval. 
- `input · number · "Voltage"`- Voltage, edit text, 12.00 
- `checkbox · switch · "Output 1"`- Output 1, switch, on 
- `image · alt "Wiring diagram"`- Wiring diagram, image
