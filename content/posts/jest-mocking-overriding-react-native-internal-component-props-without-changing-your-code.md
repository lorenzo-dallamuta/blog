## Jest Mocking: Overriding Internal Component Props Without Changing Your Code
Need to modify a React Native component's internal behavior for testing? Instead of exposing test-specific props through your component API, mock the dependency directly.

Sometimes tests need to control internal behavior of third-party components without modifying your production code. This pattern lets you override any prop or implementation detail while keeping the original component as a base.

```javascript
jest.mock('react-native', () => {
  const ActualRN = jest.requireActual('react-native');
  // Use defineProperty to properly override getter-based exports
  Object.defineProperty(ActualRN, 'FlatList', {
    value: (props) => <ActualRN.FlatList {...props} initialNumToRender={props.data?.length} />,
    writable: true,        // Allows Jest to overwrite this mock if needed
    configurable: true     // Essential: lets Jest reset mocks between tests
  });
  return ActualRN;         // Preserves all other React Native components
});
```
The key is Object.defineProperty, which safely overrides components exported as getters (like FlatList) while preserving React Native's module structure. Unlike direct assignment that fails silently on getter properties, this method properly replaces the component definition.

This approach works for any component property—whether it's `initialNumToRender` for FlatList, `scrollEnabled` for ScrollView, or any other internal behavior. You get complete control over the component's implementation while maintaining the original functionality as a foundation.