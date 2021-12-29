# React-Native-Guide
Notes For React Native

## Navigation
1. What happens when we navigate away from a route, or when we come back to it? How does a route find out that a user is leaving it or coming back to it?
If you are coming to react-navigation from a web background, you may assume that when user navigates from route A to route B, A will unmount (its componentWillUnmount is called) and A will mount again when user comes back to it. While these React lifecycle methods are still valid and are used in react-navigation, their usage differs from the web.
2. Consider a native stack navigator with screens A and B. After navigating to A, its componentDidMount is called. When pushing B, its componentDidMount is also called, but A remains mounted on the stack and its componentWillUnmount is therefore not called. When going back from B to A, componentWillUnmount of B is called, but componentDidMount of A is not because A remained mounted the whole time.
3. createNativeStackNavigator is a function that returns an object containing 2 properties: Screen and Navigator. Both of them are React components used for configuring the navigator. The Navigator should contain Screen elements as its children to define the configuration for routes. NavigationContainer is a component which manages our navigation tree and contains the navigation state. This component must wrap all navigators structure. Usually, we'd render this component at the root of our app, which is usually the component exported from App.js.
```
function DetailsScreen() {
  return (
    <View style={{ flex: 1, alignItems: 'center', justifyContent: 'center' }}>
      <Text>Details Screen</Text>
    </View>
  );
}

const Stack = createNativeStackNavigator();

function App() {
  return (
    <NavigationContainer>
      <Stack.Navigator initialRouteName="Home">
        <Stack.Screen name="Home" component={HomeScreen} />
        <Stack.Screen name="Details" component={DetailsScreen} />
      </Stack.Navigator>
    </NavigationContainer>
  );
}
```
4. The component prop accepts component, not a render function. Don't pass an inline function (e.g. component={() => <HomeScreen />}), or your component will unmount and remount losing all state when the parent component re-renders. See Passing additional props for alternatives.
5. Sometimes we might want to pass additional props to a screen. We can do that with 2 approaches:

    1. Use React context and wrap the navigator with a context provider to pass data to the screens (recommended).
    2. Use a render callback for the screen instead of specifying a component prop:
```
<Stack.Screen name="Home">
  {props => <HomeScreen {...props} extraData={someData} />}
</Stack.Screen>
```
Note: By default, React Navigation applies optimizations to screen components to prevent unnecessary renders. Using a render callback removes those optimizations. So if you use a render callback, you'll need to ensure that you use React.memo or React.PureComponent for your screen components to avoid performance issues.
6. 
