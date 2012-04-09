---
layout: post
title: "UIViewController Life Cycle"
date: 2012-04-08 17:03
comments: false
categories: [ios]
author: Alex Moffat <alex.moffat@gmail.com>
---

There are many resources describing the [life cycle of a view controller in iOS](http://developer.apple.com/library/ios/#featuredarticles/ViewControllerPGforiPhoneOS/ViewLoadingandUnloading/ViewLoadingandUnloading.html). This one differs in looking at the order of the the life cycles events of two UIViewControllers embedded in a UINavigationController. The way in which the events on the two controllers are interleaved is interesting.

The main events are

* (void)viewWillAppear:(BOOL)animated
* (void)viewDidAppear:(BOOL)animated
* (void)viewWillDisappear:(BOOL)animated
* (void)viewDidDisappear:(BOOL)animated

but I've also thrown in the [UINavigationControllerDelegate](http://developer.apple.com/library/ios/#documentation/uikit/reference/UINavigationController_Class/Reference/Reference.html) methods and some of the [UITextFieldDelegate](http://developer.apple.com/library/ios/#documentation/uikit/reference/UITextFieldDelegate_Protocol/UITextFieldDelegate/UITextFieldDelegate.html) methods.

<!-- more -->

The layout of the interface is shown below.

![Layout](/images/2012-04-08-interface-layout.png)

Pressing button B on the first screen A invokes a segue to go to screen B. On B there is a text field. The use case I experimented with was

* Start app
* Tap button B
* Tap text field on B screen
* Tap back button in navigation bar without dismissing keyboard first.
* Close app.

The sequence of events is shown as the output of NSLog statements I added to the code.

``` text Sequence of life cycle events
 <A: 0x6c11b10> viewDidLoad
 <A: 0x6c11b10> viewWillAppear
 <UINavigationController: 0x6c118f0> willShowViewController <A: 0x6c11b10>
 <UINavigationController: 0x6c118f0> didShowViewController <A: 0x6c11b10>
 <A: 0x6c11b10> viewDidAppear
 <UINavigationController: 0x6c118f0> didShowViewController <A: 0x6c11b10>
 <A: 0x6c11b10> prepareForSegue from <A: 0x6c11b10> to <B: 0x6c16980>
 <B: 0x6c16980> viewDidLoad
 <A: 0x6c11b10> viewWillDisappear
 <B: 0x6c16980> viewWillAppear
 <UINavigationController: 0x6c118f0> willShowViewController <B: 0x6c16980>
 <A: 0x6c11b10> viewDidDisappear
 <B: 0x6c16980> viewDidAppear
 <UINavigationController: 0x6c118f0> didShowViewController <B: 0x6c16980>
 <B: 0x6c16980> textFieldShouldBeginEditing
 <B: 0x6c16980> textFieldDidBeginEditing
 <B: 0x6c16980> viewWillDisappear
 <A: 0x6c11b10> viewWillAppear
 <UINavigationController: 0x6c118f0> willShowViewController <A: 0x6c11b10>
 <B: 0x6c16980> textFieldShouldEndEditing
 <B: 0x6c16980> textFieldDidEndEditing
 <B: 0x6c16980> viewDidDisappear
 <A: 0x6c11b10> viewDidAppear
 <UINavigationController: 0x6c118f0> didShowViewController <A: 0x6c11b10>
```

Notes.

* I added A as the UINavigationControllerDelegate in A's viewDidLoad method. Interestingly you see two occurrences of didShowViewController for A, one on line 4 and one on line 6. I think the first one, on line 4 is not correct as A's viewDidAppear method has not yet executed.
* When the button is pressed the first method called is prepareForSegue (line 7). At this point B exists but its view has not yet been loaded. If you're using segues then you're probably going to be calling methods on the controller you're seguing to, remember that that controller has no view when you're doing this.
* The sequence of events on lines 8 through 14 is interesing. First B's view is loaded, then A is told it will disappear. Next, before A actually does disappear B is told it will appear. Then the navigation controller delegate is told that B will be shown and only after that is A told it has disappeared and then B that it has appeared. The point, for me at least, is that you have to think about the interleaving of events as much as the sequence. This becomes clearer when we look at the text editing sequence.
* Starting on line 17 is the sequence of events that happens when you press the back button to return from screen B while the keyboard for the text field is still showing. Notice all of the things that happen before the textFieldDidEndEditing method is called on line 21. If you're using textFieldDidEndEditing to update properties you won't have access to the updated values from the text field till viewDidDisappear. So, given that you have to use B's viewDidDisappear to access text field values if you're updating in B an object passed from A that object won't be updated till A's viewDidAppear method is called. If you use A's viewWillAppear method you'll see the un-updated values.

I suppose the larger point is that it's very easy with storyboards to set up this sort of test case to explore when the actual sequence of event is, and it's sometimes not what you expect.

