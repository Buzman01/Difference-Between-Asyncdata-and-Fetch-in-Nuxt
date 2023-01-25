# Difference-Between-Asyncdata-and-Fetch-in-Nuxt
Nuxt provides two useful hooks for fetching data: AsyncData and Fetch. They’re available at different times in the Nuxt lifecycle, affecting how we use them.
Fetching data in your application is not just about loading it but also doing so at the right time (i.e. server-side vs. client-side). ⏰

To load our application data, Nuxt provides us with two useful hooks: AsyncData and Fetch (and that is not the JS Fetch API we’re talking about). The main difference between these two hooks is that they are available at different stages of Nuxt’s lifecycle. Which, of course, has its implications, as we’ll see together.

👩🏻‍🏫 We will start by “locating” these two hooks in the Nuxt lifecycle before diving into each hook’s specificities and how to use them. Then, we will compare them to see which one is a better fit for each use case.

Nuxt Lifecycle
The main implication is that the fetch hook can be called in any component (page or UI components alike), while asyncData can only be called from page components. This means that inside fetch, the component context (this) becomes available so we are now able to mutate the data of a component directly.

Fetch
The fetch hook can be called, as the diagram above shows us:

On the server-side, when rendering the route, and
On the client-side after the component is mounted.export default {
  data() {
    return {
      articles: [],
    };
  },

  async fetch() {
    // Fetch a random list of articles
    this.articles = await fetch("https://jsonplaceholder.typicode.com/posts").then((res) => res.json());

    // You will be able to access articles anywhere with this.articles and loop them v-for inside your template
  },
};
You can also call fetch on demand using this.$fetch() anywhere inside your component (e.g. inside your watchers, your methods, etc.).

<template>
  <!-- called from the template -->
  <button @click="$fetch">Refresh</button>

  <!-- called using a method -->
  <button @click="refresh()">Refresh</button>
</template>

<script>
  export default {
    methods: {
      refresh() {
        this.$fetch();
      },
    },
  };
</script>
You can customize your fetch API calls using the following options:

fetchOnServer (Boolean / Function | defaults to true): When true, the fetch call will be initiated on the server-side as opposed to client-side only.

fetchDelay (Integer | Default to 200 milliseconds): This option sets a minimum delay time to execute our fetch call, so we don’t have a quick sort of flash when our data is loaded into the page. I don’t think you’ll need to use it, as I think that the default value is enough to avoid it.

fetchKey (String / Function | Defaults to component’s ID or name): If you need to keep track of your API calls, you can use fetchKey to provide you with a key. You can also generate a unique id through a function.

export default {
  data() {
    return {
      articles: [],
    };
  },

  async fetch() {
    this.articles = await fetch("https://jsonplaceholder.typicode.com/posts").then((res) => res.json());
  },

  fetchOnServer: false,

  // Generates a random key using a function
  // Altenatively you can set up a static key
  // fetchKey: 'homepage-article',
  fetchKey() {
    const randomKey = Math.random().toString(36).substring(7);

    return randomKey;
  },
};
JavaScript
To make your application more performant, you can cache several pages and their fetched data, adding the keep-alive directive to your nuxt template. This way, fetch will only be triggered on the first visit, and the rest of the time pages and their data will be rendered from stored memory. You can also set the number of pages to be cached using keep-alive-props.

<template>
  <nuxt keep-alive keep-alive-props="{ max: 10 }"></nuxt>
</template>
HTML
To further optimize your page performance and user experience, you can use $fetchState's properties :

$fetchState.pending is a Boolean that you can use to set a placeholder when its value is true (by the way, have you heard about Vue Content Placeholders? 👌). $fetchState.pending returns true until the data is loading.

$fetchState.error will allow you to detect errors and thus be able to display an error message.

$fetchState.timestamp is, as the name states, a timestamp of the last fetch called. Why would you need that? Well, when you’re using keep-alive, you can combine $fetchState.timestamp with the activated() hook to optimize your data caching. This way, you set up a specific number of seconds before you can call fetch again.

<template>
  <div v-if="$fetchState.pending">Placeholder</div>
  <div v-if="$fetchState.error">Error Message</div>
  <div v-else>Loaded Data</div>
</template>

<script>
  export default {
    data() {
      return {
        articles: [],
      };
    },

    activated() {
      if (this.$fetchState.timestamp <= Date.now() - 60000) {
        this.$fetch();
      }
    },

    async fetch() {
      this.articles = await fetch("https://jsonplaceholder.typicode.com/posts").then((res) => res.json());
    },
  };
</script>
HTML
AsyncData
The asyncData hook is another way to fetch data server-side. It waits for its promise to be resolved before rendering the page and directly merges the return value to the local state (you know, the data function you use in every component 😉).

export default {
  async asyncData({ $content, params }) {
    articles = await $content("https://jsonplaceholder.typicode.com/posts").then((res) => res.json());

    return {
      articles,
    };
  },
};
JavaScript
This means that:

There are no placeholders you can set up;
If an error occurs, you will be redirected to the error page instead of showing an error message on the page route you’re supposed to land in; and
As we’ve mentioned before, you can fetch data using asyncData only in page components.
You can get around this last limitation by either: 1️⃣ Fetching your data in the mounted hook, but you lose server-side rendering. 2️⃣ Or passing the data fetched through asyncData as props to the pages block components, but again, the downside this time is code readability (and thus maintainability) if you have countless data calls for each block component.


Fetch vs. AsyncData: Comparative Table
Options	Fetch Hook	AsyncData Hook
Can be called from any component	✅	❌
(only in page components)
Access to context (this)	✅	❌
Listens to query string changes	✅	❌
Fit for dynamic components
(dynamic footers and navbars, filters and sorters, etc.)	✅	❌
Caching	✅	❌
Use of a placeholder	✅	❌
(can be replaced by a loading bar when navigating from one page to another)
Detects errors	✅	❌
(can only redirect to an error page when an error occurs)
Fetch vs. AsyncData: Pertinent Use Cases
As we’ve mentioned earlier, the fetch hook will be perfectly fitting to make an API call that serves a list of articles. But this list may need to be refreshed by adding new articles and/or loading more articles as the user scrolls down.

In this example, we’ll fetch data and add it to the store. It’s the most common use case, in which we use fetch to retrieve data and update the store (or the component’s local state).

export default {
  mounted() {
    window.onscroll = (e) => {
      if ((window.innerHeight + window.scrollY) >= document.body.offsetHeight) {
        this.$fetch();
      }
    };
  }

  async fetch() {
    try {
      // Fetch articles and add them to the Vues store
      this.articles = await this.$store.dispatch('Articles/fetchArticles');

      // Or fetch articles and update the component local state
      this.articles = await axios.get('/articles');
    } catch (error) {
      const message = error.response.data.message;

      console.error(error);
    }
  }
}
JavaScript
I usually stick with fetch, but there is a case when I switch to the AsyncData hook. It is when I need to retrieve the data of a markdown file with the Nuxt Content package. Here is a quick example:

export default {
  async asyncData({ $content, params }) {
    const article = await $content("articles", params.slug).fetch();

    return { article };
  },
};
JavaScript
Conclusion
I hope that with this article you can better grasp the differences between Fetch and AsyncData. I am also always happy to read your comments
