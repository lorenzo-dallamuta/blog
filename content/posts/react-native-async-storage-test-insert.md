# How To Test an AsyncStorage Insert in React Native

In a React Native app async storage any developer would eventually encounter this issue: testing a function that saves data to `@react-native-async-storage/async-storage`. The library itself doesn't need to be tested; you want to test that your code calls the library correctly with the right data.

A typical pattern is to have a service or a wrapper that encapsulates your storage logic. Let's say you have a method like this that takes an object does things with it and finally saves it to storage:

```javascript
// DiveSiteStore.ts
async saveDiveSite(diveSite: DiveSite): Promise<string | null> {
  const diveSites = await this.getDiveSites();
  let id: string | null = null;
  
  // The key logic: if it's a new dive site, generate an ID
  if (!diveSite.id) {
    id = this.generateId(); // This creates a unique string identifier
    const newDiveSite: DiveSite = {
      ...diveSite,
      id
    };
    diveSites.push(newDiveSite);
  } else {
    id = diveSite.id
    // ... handle updates ...
  }

  // This is the line we want to test
  await AsyncStorage.setItem(DIVES_STORAGE_KEY, JSON.stringify(diveSites));
  
  return id;
}
```

The main goal of our test is simple: when we call saveDiveSite with a new object, does it call AsyncStorage.setItem with a correctly formatted string that includes our new data and a generated ID?

Here’s how to do it step-by-step.

## Step 1: Set Up the Mock
First, ensure you've mocked AsyncStorage globally, usually in a `setupFilesAfterEnv` file like `jest.setup.js`.

```javascript
// jest.setup.js
import mockAsyncStorage from '@react-native-async-storage/async-storage/jest/async-storage-mock';

jest.mock('@react-native-async-storage/async-storage', () => mockAsyncStorage);
```
This replaces the real module with a Jest mock function, allowing us to spy on its methods.

# Step 2: Write the Test

Now, in your test file, you can write a focused test for the insert functionality.

```javascript
// DiveSiteStore.test.ts
import AsyncStorage from '@react-native-async-storage/async-storage';
import { DiveSiteStore } from './DiveSiteStore';

// Mock your ID generator if necessary
jest.mock('./idGenerator', () => ({
  generateId: () => 'mock-generated-id-123'
}));

describe('DiveSiteStore - saveDiveSite (insert)', () => {
  let store: DiveSiteStore;

  beforeEach(() => {
    store = new DiveSiteStore();
    // Clear mock calls before each test
    (AsyncStorage.setItem as jest.Mock).mockClear();
  });

  it('should call AsyncStorage.setItem with the new item containing a generated ID', async () => {
    // 1. Arrange: Create a new dive site object without an ID
    const newDiveSite = {
      id: null, // This signifies a new item
      name: 'Great Barrier Reef',
      // all those other meaningful propertie go here
    };

    // 2. Act: Call the method under test
    const returnedId = await store.saveDiveSite(newDiveSite);

    // 3. Assert: Verify the interaction with AsyncStorage

    // Check that setItem was called at all
    expect(AsyncStorage.setItem).toHaveBeenCalledTimes(1);

    // Get the arguments of the first call to setItem
    const mockSetItemCalls = (AsyncStorage.setItem as jest.Mock).mock.calls;
    const [calledKey, calledValue] = mockSetItemCalls[0];

    // Check it was called with the correct key
    expect(calledKey).toBe(DIVE_SITE_STORAGE_KEY);

    // Parse the value it was called with (a JSON string)
    const savedData = JSON.parse(calledValue);

    // Find the new object in the array that was saved
    expect(savedData).toContainEqual(
      expect.objectContaining({
        ...newDiveSite,
        id: returnedId // Verify ID value
      })
    );

    expect(savedNewDiveSite.data.properties.id).toBe(returnedId); // ID was generated
  });

  // 4. Assert: Verify additional logic with the method under test
  // ...
});
```

# Why Doing It This Way

What I like about this approach is that it doesn't test the mock storage itself, but rather verifies that my method is having the right conversation with AsyncStorage. I'm checking: "Did you tell AsyncStorage to save exactly what I expected you to save?"

It spies on the dependency (AsyncStorage.setItem).

It triggers your logic.

It intercepts and analyzes the inputs (the arguments) of that dependency call.


I think this is a pretty clean way to test storage wrappers. How would you test this scenario? Would you mock things differently? I'd love to hear your thoughts in the comments.