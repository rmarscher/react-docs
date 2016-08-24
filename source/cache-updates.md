---
title: Updating your UI
order: 7
---

Assigning object identifiers to your result objects using `dataIdFromObject` allows Apollo Client to reactively tell your queries about update your queries with results when new information becomes available. But, in some cases, just using `dataIdFromObject` is not enough for your application UI to get these updates. In particular, if you want to add something to a list of objects or if there are some objects that you can't assign an object identifier to, you should use either `fetchMore` or `updateQueries` in order to make sure that the queries on your page are updated with the right information and your UI updates correctly.

## `fetchMore`

Typically, `fetchMore` is used to manually update the result of one query based on the data returned by another query. Most often, it is used to handle pagination. In our GitHunt example, we have a paginated feed that displays a list of GitHub respositories. When we hit the "Load More" button, we don't Apollo Client to throw away the repository information it has already loaded. Instead, it should just append the newly loaded repositories to the list of repositories that Apollo Client already has in the store. With this update, our UI component should re-render and show us all of the available repositories.

This is possible with `fetchMore`. The `fetchMore` method allows us to fetch another query and incorporate that query's result into the result that our component query previously received. We can see it in action within the GitHunt code:

```javascript
const FEED_QUERY = gql`
  query Feed($type: FeedType!, $offset: Int, $limit: Int) {
    // ...
  }`;
const FeedWithData = graphql(FEED_QUERY, {
  props({ data: { loading, feed, currentUser, fetchMore } }) {
    return {
      loading,
      feed,
      currentUser,
      loadNextPage() {
        return fetchMore({
          variables: {
            offset: feed.length,
          },
          updateQuery: (prev, { fetchMoreResult }) => {
            if (!fetchMoreResult.data) { return prev; }
            return Object.assign({}, prev, {
              feed: [...prev.feed, ...fetchMoreResult.data.feed],
            });
          },
        });
      },
    };
  },
})(Feed);
```

We have two components here: `FeedWithData` and `Feed`. The `FeedWithData` implementation produces the `props` to be passed to the `Feed` component which serves as the presentation layer, i.e. it produces the UI. Specifically, we're mapping the `loadNextPage` prop to the following:

```
return fetchMore({
  variables: {
    offset: feed.length,
  },
  updateQuery: (prev, { fetchMoreResult }) => {
    if (!fetchMoreResult.data) { return prev; }
    return Object.assign({}, prev, {
      feed: [...prev.feed, ...fetchMoreResult.data.feed],
    });
  },
});
```

The `fetchMore` method takes a map of `variables` to be sent with the new query. Here, we're setting the offset to `feed.length` so that we fetch items that aren't already displayed on the feed. This variable map is merged with the one that's been specified for the query associated with the component. This means that other variables, e.g. the `limit` variable, will have the same value as they do within the component query.

It can also take a `query` named argument, which can be a GraphQL document containing a query that will be fetched in order to fetch more information; we refer to this as the `fetchMore` query. By default, the `fetchMore` query is the query associated with the component, i.e. the `FEED_QUERY` in this case.

When we call `fetchMore`, it will fire the `fetchMore` query and it needs to know how to incorporate the result of the query into the information the component is asking for. This is accomplished through `updateQuery`. The named argument `updateQuery` should be a function that takes the previous result of the query associated with your component (i.e. `FEED_QUERY` in this case) and the information returned by the `fetchMore` query and combine the two.

Here, the `fetchMore` query is the same as the query associated with the component. Our `updateQuery` takes the new feed items returned and just appends them onto the feed items that we'd asked for previously. With this, the UI will update and the feed will contain the next page of items!

Although `fetchMore` is often used for pagination, there are many other cases in which it is applicable. For example, suppose you have a list of items (say, a collaborative todo list) and you have a query that fetches only items that are added to this list. Then, you don't have to refetch the whole todo list: you can just incorporate the newly added items with `fetchMore` and specifiying the `updateQuery` function correctly.

## `updateQueries`

In order to incorporate the result of a mutation
