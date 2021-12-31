# React-Native-Guide
Notes For React Native
## Components
1. A view is the basic building block of UI: a small rectangular element on the screen which can be used to display text, images, or respond to user input.
2. SafeAreaView is the component that puts the UI  within the safe area boundaries of a device. Safe Area's paddings reflect the physical limitation of the screen, such as rounded corners or camera notches.
3. Core Components are <View>, <Text>, <ScrollView>, <TextInput>. Button has a onPress prop. You can still use <> </> instead of using <View></View> for top level components. TextInput has a onChangeText prop for a change in text and a onSubmitEditing prop for a function to be called when the text is submitted.
4. The FlatList component displays a scrolling list of changing, but similarly structured, data. FlatList works well for long lists of data, where the number of items might change over time. Unlike the more generic ScrollView, the FlatList only renders elements that are currently showing on the screen, not all the elements at once.

### Fast Refresh
1. Fast Refresh is a React Native feature that allows you to get near-instant feedback for changes in your React components.
2. If you edit a module that only exports React component(s), Fast Refresh will update the code only for that module, and re-render your component. You can edit anything in that file, including styles, rendering logic, event handlers, or effects.
3. If you edit a module with exports that aren't React components, Fast Refresh will re-run both that module, and the other modules importing it. So if both Button.js and Modal.js import Theme.js, editing Theme.js will update both components.
4. Finally, if you edit a file that's imported by modules outside of the React tree, Fast Refresh will fall back to doing a full reload. You might have a file which renders a React component but also exports a value that is imported by a non-React component. For example, maybe your component also exports a constant, and a non-React utility module imports it. In that case, consider migrating the constant to a separate file and importing it into both files. This will re-enable Fast Refresh to work. Other cases can usually be solved in a similar way.
5. 
  
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
4. The component prop in the above example accepts component, not a render function. Don't pass an inline function (e.g. component={() => <HomeScreen />}), or your component will unmount and remount losing all state when the parent component re-renders.
5. Sometimes we might want to pass additional props to a screen. We can do that with 2 approaches:

    1. Use React context and wrap the navigator with a context provider to pass data to the screens (recommended).
    2. Use a render callback for the screen instead of specifying a component prop:
```
<Stack.Screen name="Home">
  {props => <HomeScreen {...props} extraData={someData} />}
</Stack.Screen>
```
Note: By default, React Navigation applies optimizations to screen components to prevent unnecessary renders. Using a render callback removes those optimizations. So if you use a render callback, you'll need to ensure that you use React.memo or React.PureComponent for your screen components to avoid performance issues.
6. ```function DetailsScreen({ navigation }) {
  return (
    <View style={{ flex: 1, alignItems: 'center', justifyContent: 'center' }}>
      <Text>Details Screen</Text>
      <Button
        title="Go to Details... again"
        onPress={() => navigation.navigate('Details')}
      />
    </View>
  );
}```
If you run this code, you'll notice that when you tap "Go to Details... again" that it doesn't do anything! This is because we are already on the Details route. The navigate function roughly means "go to this screen", and if you are already on that screen then it makes sense that it would do nothing. Let's suppose that we actually want to add another details screen. This is pretty common in cases where you pass in some unique data to each route (more on that later when we talk about params!). To do this, we can change navigate to push. This allows us to express the intent to add another route regardless of the existing navigation history. Sometimes you'll want to be able to programmatically trigger this behavior, and for that you can use navigation.goBack(). 
7. navigation.popToTop(), which goes back to the first screen in the stack.
8. Pass params to a route by putting them in an object as a second parameter to the navigation.navigate function: navigation.navigate('RouteName', { /* params go here */ }) 
9. Read the params in your screen component: route.params.
``` function HomeScreen({ navigation }) {
  return (
    <View style={{ flex: 1, alignItems: 'center', justifyContent: 'center' }}>
      <Text>Home Screen</Text>
      <Button
        title="Go to Details"
        onPress={() => {
          /* 1. Navigate to the Details route with params */
          navigation.navigate('Details', {
            itemId: 86,
            otherParam: 'anything you want here',
          });
        }}
      />
    </View>
  );
}

function DetailsScreen({ route, navigation }) {
  /* 2. Get the param */
  const { itemId, otherParam } = route.params;
  return (
    <View style={{ flex: 1, alignItems: 'center', justifyContent: 'center' }}>
      <Text>Details Screen</Text>
      <Text>itemId: {JSON.stringify(itemId)}</Text>
      <Text>otherParam: {JSON.stringify(otherParam)}</Text>
      <Button
        title="Go to Details... again"
        onPress={() =>
          navigation.push('Details', {
            itemId: Math.floor(Math.random() * 100),
          })
        }
      />
      <Button title="Go to Home" onPress={() => navigation.navigate('Home')} />
      <Button title="Go back" onPress={() => navigation.goBack()} />
    </View>
  );
} 
```
10.React Navigation emits events to screen components that subscribe to them. We can listen to focus and blur events to know when a screen comes into focus or goes out of focus respectively.
```
Example:

Try this example on Snack 
function Profile({ navigation }) {
  React.useEffect(() => {
    const unsubscribe = navigation.addListener('focus', () => {
      // Screen was focused
      // Do something
    });

    return unsubscribe;
  }, [navigation]);

  return <ProfileContent />;
}
```
Instead of adding event listeners manually, we can use the useFocusEffect hook to perform side effects. It's like React's useEffect hook, but it ties into the navigation lifecycle.
