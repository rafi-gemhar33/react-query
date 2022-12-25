
## useQuery 

```jsx
const fetchSuperHeroes = () => {
  return axios.get('http://localhost:4000/superheroes')
}
const result = useQuery('super-heroes', fetchSuperHeroes, {
    onSuccess,
    onError
    cacheTime: 30000,
    staleTime: 50000,
    refetchOnMount: true, // 'always'
    refetchOnWindowFocus: true, // 'always'
    refetchInterval: 2000,
    refetchIntervalInBackground: true,
    enable: true, 
  })
const {isLoading, data, isError, error, isFetching, refetch} = result
```
key -> first param 'super-heroes'

## ReactQueryDevtools
```
import { ReactQueryDevtools } from 'react-query/devtools'
```

### Query Cache
cacheTime 
  - in 3rd param object of useQuery
### Stale Time
 - staleTime
### refetch on mount
 - refetchOnMount
### refetch On Window Focus
 - refetchOnWindowFocus
### polling
 - refetchInterval
   - only in focus
 - refetchIntervalInBackground
 - to stop pollong use a state variable and use it as the value & set it to false to stop polling
### disable fetching
  - enable
  - use refetch to fetch again.
### Success and Error Callbacks
  - onSuccess
  - onError

## Data Transformation
  - select
```jsx
select: data => {
  const superHeroNames = data.data.map(hero => hero.name)
  return superHeroNames
}
```

## Custom Query Hook

```jsx
const fetchSuperHeroes = () => {
  return request({ url: '/superheroes' })
}

export const useSuperHeroesData = (onSuccess, onError) => {
  return useQuery('super-heroes', fetchSuperHeroes, {
    onSuccess,
    onError
  })
}
```

## useQueries
```jsx

const fetchSuperHero = heroId => {
  return axios.get(`http://localhost:4000/superheroes/${heroId}`)
}

export const DynamicParallelPage = ({ heroIds }) => {
  const queryResults = useQueries(
    heroIds.map(id => {
      return {
        queryKey: ['super-hero', id],
        queryFn: () => fetchSuperHero(id)
      }
    })
  )

  return <div></div>
}

```

## Dependent Queries
```JSX
  useQuery(['courses', channelId], () => fetchCoursesByChannelId(channelId), {
    enabled: !!channelId
  })
```
 enabled: !!channelId

##  Initial Query Data
- useQueryClient
- `queryClient` object has `.getQueryData('super-heroes')
```jsx
const queryClient = useQueryClient()
return useQuery(['super-hero', heroId], fetchSuperHero, {
  initialData: () => {
    const hero = queryClient
      .getQueryData('super-heroes')
      ?.data?.find(hero => hero.id === parseInt(heroId))
    return hero ? { data: hero } : undefined
  }
})

```

## Paginated
- keepPreviousData
```jsx
  const [pageNumber, setPageNumber] = useState(1)
  const { isLoading, isError, error, data, isFetching } = useQuery(
    ['colors', pageNumber],
    () => fetchColors(pageNumber),
    {
      keepPreviousData: true
    }
  )
```

## InfiniteQueriesPage

- useInfiniteQuery

```jsx
const fetchColors = ({ pageParam = 1 }) => {
  return axios.get(`http://localhost:4000/colors?_limit=2&_page=${pageParam}`)
}

export const InfiniteQueriesPage = () => {
  const {
    // isLoading, isError, error, data, 
    fetchNextPage, hasNextPage, isFetching, isFetchingNextPage
  } = useInfiniteQuery(['colors'], fetchColors, {
    getNextPageParam: (_lastPage, pages) => {
      if (pages.length < 4) {
        return pages.length + 1
      } else {
        return undefined
      }
    }
  })
  ...
{data?.pages.map((group, i) => { // data?.pages. is available here
  return (
    <Fragment key={i}>
      {group.data.map(color => (
        <h2 key={color.id}>
          {color.id} {color.label}
        </h2>
      ))}
    </Fragment>
  )
})}
  ...
<button onClick={() => fetchNextPage()} disabled={!hasNextPage}> Load more </button>
<div>{isFetching && !isFetchingNextPage ? 'Fetching...' : null}</div>
```

## Mutations
- useMutation
- invalidate query key by `queryClient.invalidateQueries(key)`
- queryClient.cancelQueries('keys')
- queryClient.setQueryData(
```jsx
useMutation(mutation_func)
const addSuperHero = hero => {
  return axios.post('http://localhost:4000/superheroes', hero)
}
return useMutation(addSuperHero, {
    onSuccess: data => {
      queryClient.invalidateQueries('super-heroes')
    }
      /** Handling Mutation Response Start */
    queryClient.setQueryData('super-heroes', oldQueryData => {
      return {
        ...oldQueryData,
        data: [...oldQueryData.data, data.data]
      }
    })
    },
  })
}
  const { mutate: addHero } = useAddSuperHeroData()

  const handleAddHeroClick = () => {
    const hero = { name, alterEgo }
    addHero(hero)
  }

```

## Optimistic Updates
- onSettled => 
```jsx 
// 2nd param inside 
useMutation(addSuperHero, {
onMutate: async newHero => {
    await queryClient.cancelQueries('super-heroes')
    const previousHeroData = queryClient.getQueryData('super-heroes')
    queryClient.setQueryData('super-heroes', oldQueryData => {
      return {
        ...oldQueryData,
        data: [
          ...oldQueryData.data,
          { id: oldQueryData?.data?.length + 1, ...newHero }
        ]
      }
    })
    return { previousHeroData }
  },
  onError: (_err, _newTodo, context) => {
    queryClient.setQueryData('super-heroes', context.previousHeroData)
  },
  onSettled: () => {
    queryClient.invalidateQueries('super-heroes')
  }
}
```

## Axios Interceptor
```jsx
const client = axios.create({ baseURL: 'http://localhost:4000' })

export const request = ({ ...options }) => {
  client.defaults.headers.common.Authorization = `Bearer token`
  const onSuccess = response => response
  const onError = error =>  error
  return client(options).then(onSuccess).catch(onError)
}
```