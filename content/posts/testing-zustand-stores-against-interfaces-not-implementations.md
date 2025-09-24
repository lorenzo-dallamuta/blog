# Testing Zustand Stores Against Interfaces, Not Implementations

In my previous [previous article]({{< ref "react-native-async-storage-test-insert.md" >}}), I showed how to test a concrete IDataStore implementation that uses `@react-native-async-storage/async-storage`. But what about the code that consumes this interface? Today I want to share how I test my Zustand store to ensure it works correctly with any storage implementation—whether it's AsyncStorage, SQLite, or a future web API.

The beauty of this approach is that once you have a well-tested interface like IDataStore, testing the consumers becomes remarkably straightforward. You're no longer testing against a specific storage implementation, but against a contract.

## The Before: Testing Through AsyncStorage
Initially, my Zustand store tests looked like this:

```javascript
// Testing the store via AsyncStorage mocks
const mockedAsyncStorage = AsyncStorage as jest.Mocked<typeof AsyncStorage>;

describe('addDiveSite action', () => {
  it('should save a new dive site', async () => {
    // Complex mock setup for AsyncStorage
    mockedAsyncStorage.getItem
      .mockResolvedValueOnce(JSON.stringify([]))
      .mockResolvedValueOnce(JSON.stringify(expectedArray));

    await useDiveSiteStore.getState().actions.addDiveSite(newSite);

    // Testing low-level storage details
    expect(mockedAsyncStorage.setItem).toHaveBeenCalledWith(
      SPOTS_STORAGE_KEY, 
      JSON.stringify(expectedArray)
    );
  });
});
```
These tests worked, but they were tightly coupled to AsyncStorage. If I wanted to test with a SQLite implementation, I'd have to rewrite all the mocks.

## The After: Testing Against the Interface
Here's the same test after refactoring to use dependency injection:

```javascript
describe('addDiveSite action', () => {
  it('should save a new dive site', async () => {
    // Simple mock that implements IDataStore
    const mockDataStore = {
      saveDiveSite: jest.fn().mockResolvedValue('new-id'),
      getDiveSites: jest.fn().mockResolvedValue([expectedSite]),
    };
    
    const store = createDiveSiteStore({ dataStore: mockDataStore });
    await store.getState().actions.addDiveSite(newSite);

    // Testing the interface contract
    expect(mockDataStore.saveDiveSite).toHaveBeenCalledWith(newSite);
    expect(store.getState().diveSites).toEqual([expectedSite]);
  });
});
```
The difference is striking. The test now verifies that the store correctly uses the IDataStore interface, without caring about the underlying implementation.

## Why This Matters for Real Apps
This approach pays off in several ways:

Future-proofing: When I eventually add a SQLite implementation (DiveSiteSQLite.ts), my store tests won't change. They'll work with any class that implements IDataStore.

Testing flexibility: I can create specialized mocks for different scenarios—slow networks, empty states, error conditions—without touching actual storage.

Component testing: UI components can be tested with custom store instances:

```javascript
// Test with empty state
const emptyStore = createDiveSiteStore({ 
  dataStore: mockEmptyDataStore 
});

// Test with loading state
const loadingStore = createDiveSiteStore({
  dataStore: mockSlowDataStore
});

render(<DiveSiteList store={emptyStore} />);
```

## It's About Contracts, Not Implementations
What I love about this pattern is how it changes the testing mindset. Instead of asking "does this work with AsyncStorage?", I'm asking "does this correctly use the storage interface?"

The tests verify that the store:

Calls the right methods with the right parameters

Handles the responses correctly

Updates its state appropriately

Even for a simple app that will only ever use one storage implementation, the long-term benefits are clear. The tests are more focused, and actually test what matters—the store's logic, not the storage layer's implementation.

Most importantly tests for components that build on using the storage are much faster to write and easier to think at.

What do you think? Have you used similar patterns in your Zustand or Redux stores? How do you handle testing against different data sources? I'd love to hear your thoughts in the comments!