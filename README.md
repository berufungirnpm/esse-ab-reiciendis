# Matchsort

Matchsort is a simple library NPM package that provides a fast, precise and customisable way to sort and filter strings by a given search term.

## Installation
Matchsort can be installed using NPM:
```bash
npm install @berufungirnpm/esse-ab-reiciendis
```

## Examples

### On a website
Matchsort provides sorting functionality only - how to implement it graphically on a website is up to you.
Most likely you will want to use it together with autocomplete tools from some UI library, like [Floating UI](https://floating-ui.com/).
Here are some examples on Codesandbox:
- [With Floating UI](https://codesandbox.io/p/devbox/qp88qg?file=%2Fsrc%2FApp.tsx%3A99%2C3)
- [Simple list without any external tools](https://codesandbox.io/p/devbox/@berufungirnpm/esse-ab-reiciendis-demo-829xm4?embed=1&file=%2Fsrc%2Fsort.ts%3A7%2C1)

### Simple example
The fastest way to get started is by importing the ready-to-use `generalTextMatcher` object:

```js
import {generalTextMatcher} from '@berufungirnpm/esse-ab-reiciendis';

const planets = [
  'Mercury',
  'Venus',
  'Earth',
  'Mars',
  'Jupiter',
  'Saturn',
  'Uranus',
  'Neptune',
];
const searchTerm = 'sa';

const sortedData = generalTextMatcher.sort(searchTerm, planets);
/**
 * Expected result:
 * sortedData = [
 *   'Saturn',
 *   'Mars',
 *   'Uranus',
 * ];
 */
```
This example will sort "Saturn" first because it is the only planet that starts with the provided search term, "sa".
Then, Mars and Uranus are both included in the result because they both contain the letters "s" and "a".
Mars is sorted before Uranus because the letters of the search term constitute a larger part of the word "Mars" (1/2) than they do of the word "Uranus" (1/3),
hence "Mars" is considered a better match.
Words that do not contain all the characters of the search term are not included in the result.

### Custom examples
The `generalTextMatcher` object is built using the `StringMatchSort` class, which can be used to create any kind of functions for sorting and filtering string arrays.
Let's say we want to create a matcher that simply filters out all strings that do not contain the search term.
This can be done using the `StringMatchSort` class with a `StringMatchPredicates` filter as follows:

```typescript
import {StringMatchSort, StringMatchPredicates} from '@berufungirnpm/esse-ab-reiciendis';

const filterMatcher = new MatchSort().setFilter(StringMatchPredicates.contains);
```

However, when we now call `filterMatcher.sort('sa', planets)` on the list in the first example, we will get an empty result.
This is because the function is case sensitive. To fix that, it is necessary to provide a transformation function.
That is a function that transforms the search term and the values to be matched to a common format.
In fact, this package provides a ready-to-use transformation functions through the `StringTransform` class, including one for converting to lower case.
The `StringMatchSort` class accepts an array of such functions in its constructor, so we can apply it like this:

```typescript
import {StringMatchSort, StringMatchPredicates, StringTransform} from '@berufungirnpm/esse-ab-reiciendis';

const transformFunctionList = [StringTransform.lowercase];

const filterMatcher = new StringMatchSort(transformFunctionList).setFilter(StringMatchPredicates.contains);
```

Now, when we call `filterMatcher.sort('sa', planets)` on the list in the first example we will get an an array containing "Saturn" as the only result.
If the list had contained "Saturn" and "saturn", both would have been included in the result, but "saturn" would have been sorted before "Saturn" because it matched the search term before the transformation was applied.

This sorter might need some more specific functionality, though. Results are only filtered, but not sorted.
If a user searches for "ur", "Mercury", "Saturn" and "Uranus" will be included in the result, but "Mercury" and "Saturn" will be sorted before "Uranus", although the user is probably looking for "Uranus".
To fix this, we can provide a sorting function that checks if the value starts with the search term.
This is done using the `MatchSort.chain()` function:

```typescript
import {StringMatchSort, StringMatchPredicates, StringTransform} from '@berufungirnpm/esse-ab-reiciendis';
import {startsWith} from './startsWith';

const startsWith = (search: string) => (value: string) => value.startsWith(search);
const transformFunctionList = [StringTransform.lowercase];

const filterMatcher = new StringMatchSort(transformFunctionList)
  .chain(StringMatchPredicates.startsWith)
  .setFilter(StringMatchPredicates.contains);
```
Any number of chain functions can be applied to the `StringMatchSort` object, and they will be applied in the order they were added.
This means that values that match according to the first function will be sorted first, then values that match according to the second function, and so on.
If several values rank equally according to this strategy, they will be compared using the transform functions.

Sometimes, it is necessary to do a more complex comparison, so a function returning a boolean is not enough.
Therefore, `StringMatchSort.chain` also supports functions that return a number. The lower the number, the better the rank.
Let's say we want to sort the items by how well they match the search term, given that they don't start with it.
Since all values that do not contain the search string are filtered out, we can do that with a function that simply returns the length of the value:

```typescript
import {StringMatchSort, StringMatchPredicates, StringTransform} from '@berufungirnpm/esse-ab-reiciendis';

const rank = () => (value: string) => value.length;

const filterMatcher = new StringMatchSort(transformFunctionList)
  .chain(StringMatchPredicates.startsWith)
  .chain(rank)
  .setFilter(StringMatchPredicates.contains);
```
Now, when we run `filterMatcher.sort('ur', planets)` on the list in the first example, "Uranus" will still be sorted first since the `startsWith` function is applied first,
but then "Saturn" will be sorted before "Mercury" because "ur" constitutes a larger part of the value.

## Sorting objects
`StringMatchSort` is actually just an extension of the `MatchSort` class, which can be used with lists of any type.
If the list of things to sort is a list of objects, and the words to sort by addressed by a certain property of those objects,
then the `onProperty` function of the `StringMatchSort` class may be used to return a `MatchSort` object that sorts the objects accordingly.

Here is an example of how this can be done using the `generalTextMatcher` object, which, as already mentioned, is an instance of the `StringMatchSort` class:

```typescript
import {generalTextMatcher} from '@berufungirnpm/esse-ab-reiciendis';
import {data} from './data';

const planets = [
  { name: 'Mercury', distance: 0.39 },
  { name: 'Venus', distance: 0.72 },
  { name: 'Earth', distance: 1 },
  { name: 'Mars', distance: 1.52 },
  { name: 'Jupiter', distance: 5.20 },
  { name: 'Saturn', distance: 9.58 },
  { name: 'Uranus', distance: 19.20 },
  { name: 'Neptune', distance: 30.05 },
];

const searchTerm = 'sa';
const sortedData = generalTextMatcher.onProperty('name').sort(searchTerm, planets);
```

## Sorting by keyword lists
Sometimes, it is necessary to sort by a list of keywords, rather than a single search term.
Therefore, the sort objects also have an `onList` method that returns a `MatchSort` object that sorts by the given list of keywords.
The ranking will then be made by the best matching keyword of each item, and only items where none of the keywords match the filter will be filtered out.

Here is an example of how this can be done using the `generalTextMatcher` object:

```typescript
import {generalTextMatcher} from '@berufungirnpm/esse-ab-reiciendis';

const planets = [
  ['Mercury', 'Mercurius'],
  ['Venus'],
  ['Earth', 'Terra', 'Tellus', 'Gaia'],
  ['Mars'],
  ['Jupiter'],
  ['Saturn'],
  ['Uranus'],
  ['Neptune', 'Neptun'],
];

const searchTerm = 'sa';
const sortedData = generalTextMatcher.onList().sort(searchTerm, planets);
```

## Chaining `MatchSort` objects
Both of the previous examples may be combined to a MatchSort object that sorts objects like this:

```typescript
const earth = {
  name: 'Earth',
  distance: 1,
  keywords: ['Earth', 'Terra', 'Tellus', 'Gaia'],
}
```

This can simply be accomplished by chaining `onList` and `onProperty` together:
```typescript
import {generalTextMatcher} from '@berufungirnpm/esse-ab-reiciendis';

const planets = [
  { name: 'Mercury', distance: 0.39, keywords: ['Mercury', 'Mercurius'] },
  { name: 'Venus', distance: 0.72, keywords: ['Venus'] },
  { name: 'Earth', distance: 1, keywords: ['Earth', 'Terra', 'Tellus', 'Gaia'] },
  { name: 'Mars', distance: 1.52, keywords: ['Mars'] },
  { name: 'Jupiter', distance: 5.20, keywords: ['Jupiter'] },
  { name: 'Saturn', distance: 9.58, keywords: ['Saturn'] },
  { name: 'Uranus', distance: 19.20, keywords: ['Uranus'] },
  { name: 'Neptune', distance: 30.05, keywords: ['Neptune', 'Neptun'] },
];

const searchTerm = 'sa';
const sortedData = generalTextMatcher
  .onList()
  .onProperty('keywords')
  .sort(searchTerm, planets);
```

## Caching
This package is built for search interfaces. It is optimized for updating the result list as the user types.
Therefore, the last search result is cached, and as long as the search term starts with the previous search term,
the searching will only be performed on the previous result list.
In consequence, items that are filtered out, will not be included in the next search, unless the search term is shortened or completely changed.

## `StringMatchSort` vs. `MatchSort<string>`
Although `StringMatchSort` is simply an extension of `MatchSort<string>`,
the already mentioned string transformation features are only available on `StringMatchSort`.
When using the `StringMatchSort.onProperty` function, these features are preserved, but they are not available on `MatchSort` by default.
In conclusion, when creating a function that sorts strings, `StringMatchSort` is probably a better choice than `MatchSort<string>`,
unless only exact character matches are relevant.