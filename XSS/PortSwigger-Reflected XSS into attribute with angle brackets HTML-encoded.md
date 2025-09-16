# Lab: Reflected XSS into attribute with angle brackets HTML-encoded

## The story

- This lab contains a search box. Whenever I type something like `<script>alert(1)</script>` into it and hit enter, the input is reflected back as `&lt;script&gt;alert(1)&lt;/script&gt;`. This means the backend is aware of HTML tags and encodes them before sending the response to the client, so we cannot use the `<script></script>` tag here.
  
  looking at the page source code we can see:
  
    ```html
    <section class=search>
      <form action=/ method=GET>
        <input type=text placeholder='Search the blog...' name=search value="&lt;script&gt;alert(1)&lt;/script&gt;">
        <button type=submit class=button>Search</button>
      </form>
    </section>
    ```
   looking at the source, we find the vulnerable spot is actually inside the **value attribute** of the **input element**: `<input type=text placeholder='Search the blog...' name=search value="&lt;script&gt;alert(1)&lt;/script&gt;">` and as the lab name also suggests _Reflected XSS into attribute with angle brackets HTML-encoded_ here we’re thinking about how to break out of this value attribute and create a new attribute (maybe one that can run JavaScript for us).

- Let's look at the solution, then talk about why it worked:
  
   ` "onmouseover="alert(1) `
  
  in response we can see:

  ```html
  <section class=search>
    <form action=/ method=GET>
      <input type=text placeholder='Search the blog...' name=search value=""onmouseover="alert(1)">
      <button type=submit class=button>Search</button>
    </form>
  </section>  
  ```
  The lab was solved by: `"onmouseover="alert(1)`
    
    1.  **Terminating the existing attribute:** using a double quote `"` to break out of the `value` attribute.
    2.  **Introducing a new, malicious attribute:** injecting an event-handler attribute such as `onmouseover`.
    3.  **Providing a value for that attribute (payload):** setting the event handler’s value to JavaScript, `alert(1)`.
 
 The browser now sees an `<input>` tag with two attributes:
 ```html
<input value="" onmouseover="alert(1)" ... >
```
     
  * `value=""` (an empty search box)
  * `onmouseover="alert(1)"` (an instruction to execute `alert(1)` when a user moves their mouse over this input box)
  
 ## What is event-handler attribute in html?

  - What is attribute in html?  An **attribute** is a **value** placed **inside** the **opening tag** of an **HTML element**. Attributes provide additional information about the element or specify how the element should behave. 
  - Now let's consider this, we know that **HTML** defines the **structure and content** of a web page, **CSS** defines the **presentation** (e.g., make the button red, make the paragraph's font large) and **JavaScript** defines the **behavior and interactivity** (e.g., show a message when the button is clicked, validate the input in the box when the user presses submit).
  - **Event handlers are the bridge between HTML (the structure) and JavaScript (the behavior).**
    
     _They are special HTML attributes that tell the browser: "When a specific **event** happens to this HTML element, run this piece **JavaScript code**."_

  ### Examples Of Event Handlers :

| Event Handler | JavaScript Code Runs When... | Example of Legitimate Use |
| :--- | :--- | :--- |
| **`onclick`** | ...the element is clicked. | Submitting a form, playing a video. |
| **`onmouseover`** | ...the mouse pointer moves over the element. | Showing a preview or a tooltip. |




