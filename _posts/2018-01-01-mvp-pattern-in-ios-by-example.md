---
title: MVP Pattern In iOS By Example
layout: post
categories: [Design Patterns, English] 
---

Giving your code some structure is important. I touched what MVP is in another post with a more abstract approach, without a piece of code. Here we will see the pattern explained by building an app feature, with folder structure and code examples. This post is heavily influenced by [Iyad Agha](Iyad Agha's post)

Everything here is an advice, sharing the knowledge I have gathered through serious study. Please, if you find a better way, let me know so we can discuss it. Enjoy.

## Context

We are building an app which helps volunteers, that depends on accesing a facebook event to perform business specific actions. Right now we will only focus on giving the user access to the event.


## User Story

**- As a leader user, I want to access an active event**

### Client acceptance tests

By tests I don't want to say by any means automated tests, that will have another whole post. This are the criteria by what the client will declare completed this user story:

- To access as a leader, the user needs to be physically on the event's location
- To access as a leader, the user needs to be registered
- The event will be a Facebook Event

## Folder structure

This is how our files will look at the end. I will explain each file later in the post.

I have found that dividing your files by *segments* of the app helps to modularize it. A segment can be a menu section or a UI screen for example.

```
|-GenericDataModels
|----Event
|-GenericServices
|----FacebookGenericService
|----FirebaseGenericService
|-Volunteer
|----VolunteerViewProtocol
|----VolunteerEventViewData
|----VolunteerViewController
|----VolunteerPresenter
|----VolunteerService
```

## The View

The view is the entry point for the user, it will be seen and interacted with. We can start by defining a struct that represents exactly the information to be visualized. For now we will only show the event's name, so the user will know what he is participating in.

```swift
struct VolunteerEventViewData {
    let name: String
    
    init(name: String) {
        self.name = name
    }
}
```

Then, a protocol will represent a contract between the view and whatever that manipulates it. In this case is a presenter, but that will be discused further on the post. Having a protocol gives us the flexibility to dettach the view from the logic, by that I mean that later we can change the view to support a totally different UI framework without touching a line of code's logic.

```swift
protocol VolunteerViewProtocol {
    
    // MARK: - Display data
    func display(data: VolunteerEventViewData)
    func displayNoData()
    func displayNoNetwork()
    
    // MARK: - Response from presenter
    // To access as a leader, the user needs to be registered.
    func requestRegistration()
    // To access as a leader, the user needs to be at the event
    func requestLocationPermission()
    
    // MARK: Leader access resolution
    func accessLeaderGranted()
    func accessLeaderDenied()
}
```

After that we can build our `ViewController` implementing the above protocol

```swift
class VolunteerViewController: UIViewController, VolunteerViewProtocol {
    
    // MARK: - Display data
    func display(data: VolunteerEventViewData) {
    	// ...show the events name
    }
    func displayNoData() {
    	// ...show that there is no data
    }
    func displayNoNetwork() {
    	// ...show that there is no data nor network available
    }
    
    // MARK: - Response from presenter
    // To access as a leader, the user needs to be registered.
    func requestRegistration() {
    	// ...alert the user to sign up
    }
    // To access as a leader, the user needs to be at the event
    func requestLocationPermission() {
    	// ...request location 
    }
    
    // MARK: Leader access resolution
    func accessLeaderGranted() {
    	// ...give access to the user
    }
    func accessLeaderDenied() {
    	// ...alert the user is far from the event's location
    }
}
```

As you can see, the view is dumb. It shouldn't make any choices, ideally not a single `if` should be present at the view.

To explain how the view wires up with the presenter, lets see where it happens. I like to do it in the `viewDidLoad()` method, like this:

```swift
// MARK: - Properties
private var presenter: VolunteerPresenter!
    
// MARK: - ViewController    
 override func viewDidLoad() {
        super.viewDidLoad()
        self.presenter = VolunteerPresenter(withService: VolunteerService())
        self.presenter.attach(view: self)
        self.presenter.load()
 }
```

## The Presenter

You already saw the first presenter methods we need, but there are plenty more. Remember that this represents most of the business logic our application needs. Something to keep in mind, is that the presenter shouldn't have any external dependencies (except that they are part of the business logic). Here in iOS the presenter shouldn't use any UIKit property or method. The only thing it knows about the presentation layer is that it is interacting with a `VolunteerViewProtocol`. This is how our file looks like:

```swift
class VolunteerPresenter: NSObject, CLLocationManagerDelegate {

	//MARK: - Properties
	private var view: VolunteerViewProtocol!
	private let service: VolunteerService
	private var event: Event!
	private var locationManager: CLLocationManager!
	    
	init(withService service: VolunteerService) {
		self.service = service
	}
	
	func attach(view: VolunteerViewProtocol){
	        self.view = view
	}
	    
	func dettachView(){
	    self.view = nil
	}
	
	func load(){
		// ... load what this segment needs
		// In our case it is to call the service to get today's event
		self.service.getEventOfToday(since: todayString, until: untilString, callback: show(data:))
	}
	
	func show(data: Event?){
		guard data != nil else {
	        self.view.displayNoData()
	        return
	   }
	   
	   self.event = data
		let viewData = VolunteerEventViewData(name: self.event.name ?? "Event")
		self.view.display(data: viewData)
	}
	
	// MARK: - Business Logic
	    
	// A leader needs to be registered in the system.
	// When a leader tries to access an event, we need to check that he is indeen at the event's location.
	// For more information check self.verify(userLocation:)
	func leaderAccessEvent() {
		//... we check if the user has signed up, otherwise, we call:
		self.view.requestRegistration()
		
		//... we check if the user hasn't denied the location services, otherwise we ask for it
		self.view.requestLocationPermission()
		
		//... if everything works as it should, we start asking to update the user location
		locationManager.requestWhenInUseAuthorization()
	}
	
	// Here we will verify that the user is at the event's location. This is called at the Location Manager Delegate
	func verify(userLocation: CLLocation) {
		//... if the user is at the event's location we call
		self.accessLeaderGranted()
		
		//... otherwise we call
		self.accessLeaderDenied()
	}
	
	func leaderAccessGranted() {
		 //... other logic to grant access to the user
        self.view.accessLeaderGranted()
    }
    
    func leaderAccessDenied() {
    //... other logic to denied access to the user
        self.view.accessLeaderDenied()
    }
    
    // MARK: - Location Manager Delegate
    func locationManager(_ manager: CLLocationManager, didUpdateLocations locations: [CLLocation]) {
    	self.verify(userLocation: userLocation)
       manager.stopUpdatingLocation()
    }
```

Now you can see in a practical way that the interaction between the view and the presenter is to call methods from each other. The presenter never access to any view's properties, or any other UIKit dependency. We also recieve at the `init(withService: VolunteerService)` method the service which the presenter will use. We do that so we can test it mocking the service. Personally I didn't choose to create a protocol for the presenter[[1]](#references) for a few reasons:

1. The presenter has no external dependencies (like the view and UIKit), so we can simply mock it and override its methods
2. It bloats the amount of code we need to mantain, and right now I haven't seen the need for it
3. it is mostly a personal choice, I am trying to find a balance between structuring in a proper way the code without forgetting that what matters is getting things done.

## The Model

As we said before, the model only cares about how data is consumed and stored. Thanks to the *client acceptance tests* we know that the event will come from Facebook, so we will create the next file:

```swift
// VolunteerService.swift

class VolunteerService {

	let fbService: FacebookProtocol
	let firService: FirebaseProtocol
    
    init(fbService: FacebookProtocol = FacebookGenericService(), firService: FirebaseProtocol = FirebaseGenericService()) {
        self.fbService = fbService
        self.firService = firService
    }
    
	// Params:
    // - since: a date in strtodate format
    // - until: a date in strtodate format
    // - callback with (Events?): Function to be executed after the method execution
    // This method makes a Facebook Graph request returning 1 event between the dates `since` and `until`
    func getEventOfToday(since: String, until: String, callback: @escaping (Event?) -> ()) {
    	// function logic...
    }
}

```

Also we need the struct to store facebook's event data:

```swift
struct Event {
    
    let id: String?
    let cover: Cover?
    let description: String?
    let name: String?
    //... other properties
}
```

Let's breake it down.

```swift
	let fbService: FacebookProtocol
	let firService: FirebaseProtocol
```

Because the app depends on facebook data, it is possible that we will need the same information on several places, so it would be wrong to define it only in `VolunteerService.swift`. This is why I made `FacebookGenericService.swift`. Right now it only looks like this:

```swift
class FacebookGenericService: FacebookProtocol {
    let pageId = "FACEBOOK PAGE ID"
}
```

You may be wondering, what is `FacebookProtocol`? Modularity is important, and separating the model from everything else let us reuse it, replace it, and test it correctly. There is a debate whether you should create protocols for something else than the view[[1]](#references), but I felt like doing it.

We also have `FirebaseGenericService.swift` which implements the `FirebaseProtocol`. This file exists because we need a token to make Graph Api requests, and I use a private one. It is a bad practice to hard code it in the application because there are ways to decompile the app and look directly at the source code. Also the facebook token expires after a few days, so it is easier to generate it automatically in a firebase function. The protocol looks like this:

```swift
protocol FirebaseProtocol {
    func getFacebookToken(callback:@escaping (String?) -> ())
}
```

Then we have the constructor. Here we receive both facebook and firebase services as parameters. This is to make dependency injection[[2]](#references), maybe in a simple way but useful for later testing. Lets see how this works:

```swift
// Good practice
init(fbService: FacebookProtocol = FacebookGenericService(), firService: FirebaseProtocol = FirebaseGenericService()) {
        self.fbService = fbService
        self.firService = firService
    }
```

This is different from the code below, because the **dependencies** are **injected**, not initialized inside the constructor, black boxing them making it rather difficult to test.

```swift
// Bad practice
init() {
        self.fbService = FacebookGenericService()
        self.firService = FirebaseGenericService()
    }
```

Another thing worth mentioning is that I gave default values to them. This let us later in the code, initialize it without the need to pass everytime an instance of the dependencies needed, this means we can use this:
```
VolunteerService()
```

Instead of:

```swift
VolunteerService(fbService: FacebookGenericService(), firService: FirebaseGenericService())
```

At the same time, if we need to change the needed dependencies, we could do it seamlessly without changing every instance creation.

At the end, we have the function that will return us what we seek. The comments make it self explanatory. I will commit the actual function logic because it is not today's topic.

```swift
// Params:
    // - since: a date in strtodate format
    // - until: a date in strtodate format
    // - callback with (Events?): Function to be executed after the method execution
    // This method makes a Facebook Graph request returning 1 event between the dates `since` and `until`
    func getEventOfToday(since: String, until: String, callback: @escaping (Event?) -> ()) {
        // function logic...
    }
```

## Wrapping up

We saw today a way to implement the mvp pattern on a more concise example, stretching it to cover edge cases where common logic is shared between user stories. `FacebookGenericService` is an example of that. We also discussed that the view must implement a protocol to decouple the presentation layer from the *backend* of the app. I covered the topic briefly about *dependency injection* and how we can use it.

If you found it useful, share the knowledge. Otherwise, if you think something can be improved please let me know in the comments.


## References

[1] http://blog.karumi.com/interfaces-for-presenters-in-mvp-are-a-waste-of-time/

[2] https://stackoverflow.com/questions/130794/what-is-dependency-injection

{% include disqus.html %}