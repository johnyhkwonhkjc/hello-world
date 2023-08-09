## Introduction
What is react-query? It is a library for fetching data in a React application. Since React is a UI library, there is no specific pattern for data fetching. If data is needed in React, we usually use state management libraries. However, these state management libraries are good for working with client state, not with working with asynchronous or server state. This is where react-query comes into place. React-query is a library that **ease the process of caching, deduping multiple requests for the same data, updating stale data in the background, and performance optimizations**. 

In this article, we will discuss about what useQuery in react-query is, and some important basic options that are used with useQuery.

## What is useQuery?
By using react-query's useQuery, we do not have to manage state variables like isLoading, data, and error based on useEffect. It makes us to write less code, since react-query's useQuery returns flags that we could use. For example, the normal way of writing a fetch request in react would be 

`import React, { useState, useEffect } from 'react';

function Todos() {
  const [isLoading, setIsLoading] = useState(true);
  const [isError, setIsError] = useState(false);
  const [data, setData] = useState([]);
  const [error, setError] = useState(null);

  useEffect(() => {
    fetchTodoList()
      .then((result) => {
        setData(result);
        setIsLoading(false);
      })
      .catch((error) => {
        setError(error);
        setIsError(true);
        setIsLoading(false);
      });
  }, []);

  if (isLoading) {
    return <span>Loading...</span>;
  }

  if (isError) {
    return <span>Error: {error.message}</span>;
  }

  return (
    <ul>
      {data.map((todo) => (
        <li key={todo.id}>{todo.title}</li>
      ))}
    </ul>
  );
}

export default Todos;`

However, by using react-query, we can make the code a lot more simple.
`function Todos() {
  const { isLoading, isError, data, error } = useQuery({
    queryKey: ['todos'],
    queryFn: fetchTodoList,
  })

  if (isLoading) {
    return <span>Loading...</span>
  }

  if (isError) {
    return <span>Error: {error.message}</span>
  }

  // We can assume by this point that `isSuccess === true`
  return (
    <ul>
      {data.map((todo) => (
        <li key={todo.id}>{todo.title}</li>
      ))}
    </ul>
  )
}`

Another advantage of using react-query is that it provides query cache by default. Every query result is cached for five minutes by default (use `cacheTime` to modify it). The first time use query is fired for a unique key, isLoading is set to true and a network request is sent to fetch data. When the request is completed, it is cached using the query key and the query function as unique identifiers. Now when we revisit the page that uses this query to fetch data, react-query will check if the data for this query exists in the cache. If it exists, the cached data is immediately returned without having the need of sending a network request. 

React-query also knows that the server data might have updated and the cache might not contain the latest data. So, a background refetch is triggered for the same query and if the fetch is successful, the new data is updated in the UI and cache. React Query automatically checks if the data has become stale based on the staleTime option (default is 0 milliseconds). If the data has become stale, React Query triggers a background refetch to update the data.

`
  const { isLoading, isError, data, error } = useQuery({
    queryKey: ['todos'],
    queryFn: fetchTodoList,
    {
	cacheTime: 5000, //by default
     }
  })
`

For example, when a data gets updated in the backend, react-query will display the original data first in the cache, and then when the list gets updated in the background, the user will see the updated data without having to see the loading indicator every single time. 

We can also improve the performance using stale time in react-query. State time could be used for data that does not changes often and stale data that is okay to see for a while. We can manually change this by using the `staleTime`. The default is 0 milliseconds.

`
  const { isLoading, isError, data, error } = useQuery({
    queryKey: ['todos'],
    queryFn: fetchTodoList,
    {
	staleTime: 30000 //set to 30sec
     }
  })
`
Now, let’s take a look at refetch in react-query. By default, the query will refetch on mount if the data is stale.
-	False : Won’t refetch data
-	Always : Will always refetch data even though the data is not stale

Another option that is slightly more important is `refetchOnWindowFocus`. By default it is true. Any time your tab loses focus and gains focus again, a background refetch is initiated. When the refetch completes, the UI is updated with the data retrieved.
-	False : Won’t refetch data
-	Always : Will always refetch data even though the data is not stale
`
  const { isLoading, isError, data, error } = useQuery({
    queryKey: ['todos'],
    queryFn: fetchTodoList,
    {
	refetchOnMount: true // by default
	refetchOnWindowFocus: true //by default
     }
  })
`
Now, lets look at polling. Polling refers to the process of fetching data at regular intervals. This could be used with real-time stock prices. You would want to fetch data every second to update the UI. This ensures that the UI will always be in sync with the remote data irrespective of configurations like refetchOnMount and refetchOnWindowFocus. We can use the option `refetchInterval`. By default it is set to false. You can set milli seconds for it. One important thing to remember here is that polling or automatic refetching is paused when window loses focus. If you want background refetching at regular intervals, you can use the option `refetchIntervalInBackground` and set it to true. It’s amazing how react-query enables us to do complicated things by just adding two lines of code. 
 
`
  const { isLoading, isError, data, error } = useQuery({
    queryKey: ['todos'],
    queryFn: fetchTodoList,
    {
	refetchInterval: 2000,
	refetchIntervalInBackground: true,
     }
  })
`
##useQuery on Click
We can also query data only when we click a button. First, we disable the fetchOnMount by setting `enabled` to false. Second, we fetch data on click of a button. useQuery returns a function called refetch to manually trigger the query. All we have to do is pass this function to the onClick handler `onClick={refetch}`.
`
  const { isLoading, isError, data, error } = useQuery({
    queryKey: ['todos'],
    queryFn: fetchTodoList,
    {
	enabled: false
     }
  })
`
##Success and Error Callbacks
You can use the `onSuccess` and `onError` option in useQuery. Also, you can use the data fetched inside the onSuccess functions and you can also use the error message inside the onError function. 
`
  const { isLoading, isError, data, error } = useQuery({
    queryKey: ['todos'],
    queryFn: fetchTodoList,
    {
	onSuccess: onSuccess // function that runs onSuccess
	onError: onError // function that runs onError
     }
  })
`
##Data Transformation
The `select` option allows us to transform the data. It receives the api data as an argument and we can use that data to return transformed data. 
`
  const { isLoading, isError, data, error } = useQuery({
    queryKey: ['todos'],
    queryFn: fetchTodoList,
    {
	select: (data) => {
		const toDoListName = data.map((todo) => todo.name)
		return toDoListName
}
     }
  })
`
