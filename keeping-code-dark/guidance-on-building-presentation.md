# Guidance on creating this presentation

In the past couple of months, we have had some features that have made it out to users in a partially built state. We've also run into places where we build separate pieces of one larger solution and have a hard time bringing it together at the end. Since we have a lot of different squads on the team and haven't completely landed feature flagging and "dark code" patterns, we're wanting to give a presentation and start the conversation.

1. Picture of my family. People listen to you more when you show them a picture of your family. (literally say that in the presentation)
2. Joke about how "dark code" is really just code that you're viewing from a light mode editor. Other joke ideas?
3. Present the idea of dark code. What do we mean by that? What are some examples?
4. Approaches on web
   1. Dark route
   2. Debug page to show a new page or flow.
   3. Debug menu feature flag.
5. Differences on mobile
   1. Optimizely is not safe for keeping dark while building, as a partially built version could show to users if you have a release in the middle of building the feature and then you turn it on later. Can use for releasing, but not for keeping dark while building.
   2. Steven's opinion - if you have UI work that you need to see as you build, trigger it from the debug page.
      1. Put it on a debug page
      2. Or put a feature flag in the debug menu
6. General patterns
   1. Indiana Jones Swap (TM) - If you're migrating from one pattern to another, you can program to a common interface and then do the classic "Indiana Jones Swap" at the end. As you're in the process of building out the replacement, it's common practice and good etiquette to refer to the the new version as "The New Hotness"

Testing Trophy

1.  Talk about the mindset of the testing trophy and how that mindset can help us here. In a scenario where we're building all of the necessary pieces for a larger feature and then wiring them together well into the process, it's really easy to drift a little bit and cause the wire-up to become a huge pain. And it's hard to know whether the main behaviors that you're going for are going to be intact. But if you get IT tests in place as early as possible that validate the key behaviors, it allows you to refactor and rework whatever layers you want while maintaining confidence that you haven't broken the behavior. e.g. In backend work, start by building out a rough working version of a service that has some IT tests around it (ones that go all the way back to the database)
2.  In explaining this, we probably want to have an image or diagram of the testing trophy (I can provide that if needed). Also want to have a diagram explaining the layers of a service application. Very lightweight controller that only cares about HTTP concerns and mapping into what the service layer expects. Service is also fairly lightweight. Orchestrates business logic, error handling, general logic flow. Avoids getting into details of model structure, mapping from one shape to another, or other things that can easily be put into a model method or mapper class and much more easily unit tested there. How do you know your service class is doing too much? Look for long methods or lots of private methods. -- Repositories are also fairly light. They understand how to interact with persistence layer and possibly how to map from database entities to domain objects (Is this right, from a hexagonal perspective???)
3.  On the subject of large service, may also want to talk generically about OOP. Lots of private methods in a service is a code smell because it's often breaking basic OOP principle of objects encapsulating data and operations on that data.
