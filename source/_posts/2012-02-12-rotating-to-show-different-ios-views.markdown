---
layout: post
title: "Rotating to show different iOS views"
date: 2012-02-12 20:11
comments: true
categories: iOS
author: Alex Moffat <alex.moffat@gmail.com>
---

In the tradition of explaining fairly simple things at great length this approach is based on the documentation from Apple describing [Creating an Alternate Landscape Interface](https://developer.apple.com/library/ios/#featuredarticles/ViewControllerPGforiPhoneOS/RespondingtoDeviceOrientationChanges/RespondingtoDeviceOrientationChanges.html#//apple_ref/doc/uid/TP40007457-CH7-SW1) and various answers on [stackoverflow](http://stackoverflow.com/). It differs from the information I could find on the internet because the flow includes a navigation controller and uses a storyboard. The flow between the screens in the interface is shown in the diagram below.

![Flow](images/2012-02-12-ios-views.png)

There is a first screen, A, that has two alternate views, one portrait and one landscape. These can't be created using the springs and struts layout tools. Rotating the  phone while screen A is showing should switch between the two views. Tapping some element on screen A, in either view, transitions to screen B. Screen B handles rotation itself by redrawing its interface in drawRect:. Going back from B should show the correct version of A. There's a navigation controller wrapping the whole interface that provides the navigation bar and button in view B to go back to view A.

The implementation uses the navigation controller's pushViewController:animated: and popViewControllerAnimated: methods to display the portrait and landscape views. There is a single parent class, AViewController, containing common functionality, with two subclasses, APortraitViewController and ALandscapeViewController. XCode can be used to layout the different portrait and landscape views. The initial state is in portrait orientation (applications always start in portrait orientation and are sent rotation messages if required) so the root view controller for the navigation controller is APortraitViewController. When the orientation changes to landscape ALandscapeViewController is pushed and when it changes back to portrait ALandscapeViewController is popped. APortrait is never removed it's just hidden by ALandscape.

![Push and Pop](images/2012-02-12-ios-views-nav.png)

Before looking at the code to push and pop the views we have to  consider how to deal with view B, and what the storyboard looks like. View B presents some problems because you can get get to view B from the landscape version of view A. While showing view B you can rotate to portrait, and press the back button, at which point you need to go back to A. At this point you want the APortrait view shown. This is the path, 1, 2, 3, 4 in the diagram below.

![Rotating in view B](images/2012-02-12-ios-views-rotate.png)

The solution to this is quite simple. At appropriate times, to be described below, check whether the portrait view is being show in landscape orientation, or the landscape view in portrait orientation and perform the appropriate push or pop. 

Finally, the last thing before the code, the picture below shows how the storyboard laid out.

![Storyboard layout](images/2012-02-12-ios-views-storyboard.png)

The ALandscape view controller is not connected to APortrait. It's given an identifier that's used to load it from code. Both APortrait and ALandscape use push segues to go to B.

Here's the code that swaps the portrait and landscape versions of the A view controller. It's in the APortraitViewController class, because an instance of that class is always in memory, so self refers to APortraitViewController. Instances of the ALandscapeViewController are created as needed for pushing and then destroyed when they are popped. It's important to set the model and any other properties for the landscape instance each time one is created.

``` objective-c Swapping view controllers
- (void)swapControllersIfNeeded 
{    
    UIDeviceOrientation deviceOrientation = [UIDevice currentDevice].orientation;
        
    // Check that we're not showing the wrong controller for the orientation.
    if (UIDeviceOrientationIsLandscape(deviceOrientation) && 
        self.navigationController.visibleViewController == self) 
    { 
        // Orientation is landscape but the visible controller is this one,
        // which is the portrait one.
        // Create new instance of landscape controller from the storyboard.
        // Use a property to keep track of it because we need it for
        // the check in the else branch.
        self.landscapeViewController = 
            [self.storyboard instantiateViewControllerWithIdentifier:@"LandscapeViewController"];
        // TODO - Set the landscape controller's model from the model we have.
        // self.landscapeViewController.model = self.model;
        // Push the new controller rather than use a segue so that we can do it 
        // without animation.
        [self.navigationController pushViewController:self.landscapeViewController 
                                             animated:NO];             
    } 
    else if (UIDeviceOrientationIsPortrait(deviceOrientation) && 
             self.navigationController.visibleViewController == self.landscapeViewController) 
    {
        // Orientation is portrait but the visible controller is
        // the landscape controller. Pop the top controller, we
        // know the portrait controller, self, is the next one down.
        [self.navigationController popViewControllerAnimated:NO];
    }
}
```

There are two occasions when swapControllersIfNeeded must be called. First, when the orientation changes, which will handle the case where ALandscape or APortrait is showing and must be swapped. Second, when the navigation view controller shows a new view controller. This is to take care of the case where rotation takes place while view B is showing.

To deal with orientation changes register, in awakeFromNib, for orientation change events and implement a method handle them.

``` objective-c Responding to orientation changes.

    // In awakeFromNib
    [[UIDevice currentDevice] beginGeneratingDeviceOrientationNotifications];
    [[NSNotificationCenter defaultCenter] addObserver:self
                                             selector:@selector(orientationChanged:)
                                                 name:UIDeviceOrientationDidChangeNotification
                                               object:nil];

// Method to handle orientation changes.
- (void)orientationChanged:(NSNotification *)notification 
{
    [self swapControllersIfNeeded];
}
```

For the navigation controller case APortraitViewController must conform to the UINavigationControllerDelegate protocol and implement one of its methods.

``` objective-c Responding to navigation controller view changes

    // In awakeFromNib register as the delegate.
    self.navigationController.delegate = self;

// Called when a new view is shown.
- (void)navigationController:(UINavigationController *)navigationController 
       didShowViewController:(UIViewController *)viewController 
                    animated:(BOOL)animated 
{
    // May be coming back from another controller to find we're
    // showing the wrong controller for the orientation.
    [self swapControllersIfNeeded];
}
```

One final wrinkle is that we don't want a back button appearing at the top of the landscape view pointing back to the protrait view. To prevent this the Shows Navigation Bar property for the Navigation Controller must be unchecked. However, we do want a back button to go from view B back to view A so in the viewWillAppear: method of the B view controller the navigation bar is unhidden. If you want APortrait and ALandscape views can have title bars added to them so their appearance is as it would be if Shows Navigation Bar was checked.

``` objective-c Showing the navigation bar
- (void)viewWillAppear:(BOOL)animated 
{
    [self.navigationController setNavigationBarHidden:NO
                                             animated:animated];
}
```
