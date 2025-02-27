{% set title = "Server Side Rendering" %}
<span id="title" class="d-none">{{ title }}</span>

<frontmatter>
  title: "{{ title }}"
  layout: devGuide.md
  pageNav: default
</frontmatter>

# {{ title }}

<div class="lead">

MarkBind makes use of Server-side Rendering (SSR) for its pages. 

To ensure SSR works properly, there are certain rules that developers should adhere to. 

This page will describe SSR in general and elaborate on the things that developers should take note of when contributing to MarkBind. 
</div>

## Pre-requisite Knowledge

To deal with SSR, it is important to first have a good understanding of two things:
1. Vue
2. MarkBind's Packages

### Understanding Vue

Here is a short list of questions to check your understanding of Vue:
- What is a Vue instance?
- What does it mean to compile Vue?
- What are render functions?
- Are there any differences between compiling Vue on client-side versus server-side?
- What is the difference between compiling and rendering?

### MarkBind's Packages

There are four packages in MarkBind's codebase:
1. cli
2. core
3. core-web
4. vue-components

You may refer to MarkBind's [project structure](projectStructure.md) to get a better understanding of how the packages work together.

## What is Server-side Rendering and Why?

In MarkBind's context, SSR refers to the generation of proper HTML strings from Vue components on the server (core library) instead of shifting that responsibility to the client-side (browser). 

The main motivation that we had for introducing SSR into MarkBind is to enhance user experience by resolving the unsightly issue of Flash-of-Unstyled-Content (FOUC), which occurs due to our reliance on Client-side Rendering (CSR). 

## Client-side Hydration

SSR and Client-side Hydration are 2 concepts that go hand-in-hand. Essentially, once we produce the static HTML via SSR and send it over to the client-side, Vue on the client-side will execute what is known as Client-side Hydration on the static HTML.

During the hydration process, Vue essentially `diff` your SSR HTML markup against the virtual DOM generated by the render function on the client-side. If any difference is found, meaning that the application that we have on the client-side (the virtual DOM tree) differs from the SSR HTML mark-up that we send to the client, Vue will reject the SSR HTML output, bail Client-side Hydration, and execute full CSR.

This is known as "Hydration Issue" and it is one of the main challenges you will face with SSR in MarkBind. 

## Penalties of Hydration Issue

When hydration fails, on top of the wasted time and effort in executing SSR, we will also incur the additional time penalty of executing Client-side Hydration (where CSR will follow afterwards).

Fortunately, even if we face hydration issues and execute full CSR, the FOUC problem will still be resolved nonetheless. The reason for this is because the SSR HTML markup should resemble the CSR HTML markup to a large extent.

Supposedly, hydration issues typically occurs due to minor differences between client-side rendered virtual DOM tree and the server-rendered content. Of course, this is assuming that we are adhering to the concept of "universal application" as much as possible, which will be explained in the following section.

## Avoiding Hydration Issue 

Conceptually, to prevent hydration issue, what we should always strive to achieve is a "universal application". 

It is not difficult to achieve a "universal application" per-se because we merely have to ensure two things:
1) the state data are the same between client-side and server-side.
2) after compiling and rendering the Vue page application, the SSR HTML mark-up is not modified.

However, beyond achieving a "universal application", there are also some more specific rules that we should adhere to, so that we do not run into hydration issue: 
- Do not violate HTML spec as much as possible (e.g. having block-level elements within `<p>` tag).
- Having unknown HTML elements within our Vue application during compilation/rendering (though this can be easily resolved by adding `v-pre` to the unknown element, so that Vue will ignore that element during compilation). 

Note that the list only included the common causes of hydration issue that MarkBind developers have ran into. There may be other causes of hydration issue that are not listed here (although unlikely).
