# Difference-Between-Asyncdata-and-Fetch-in-Nuxt
Nuxt provides two useful hooks for fetching data: AsyncData and Fetch. They’re available at different times in the Nuxt lifecycle, affecting how we use them.
Fetching data in your application is not just about loading it but also doing so at the right time (i.e. server-side vs. client-side). ⏰

To load our application data, Nuxt provides us with two useful hooks: AsyncData and Fetch (and that is not the JS Fetch API we’re talking about). The main difference between these two hooks is that they are available at different stages of Nuxt’s lifecycle. Which, of course, has its implications, as we’ll see together.

👩🏻‍🏫 We will start by “locating” these two hooks in the Nuxt lifecycle before diving into each hook’s specificities and how to use them. Then, we will compare them to see which one is a better fit for each use case.

Nuxt Lifecycle
