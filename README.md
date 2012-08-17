# SKBounceAnimation

`SKBounceAnimation` is a `CAKeyframeAnimation` subclass that creates an animation for you based on start and end values and a number of bounces. It’s based on the math and technology in this blogpost: [khanlou.com/2012/01/cakeyframeanimation-make-it-bounce/](http://khanlou.com/2012/01/cakeyframeanimation-make-it-bounce/) which in turn was based partially on Matt Gallagher’s work here: [cocoawithlove.com/2008/09/parametric-acceleration-curves-in-core.html](http://cocoawithlove.com/2008/09/parametric-acceleration-curves-in-core.html).

## Usage

Basic code is simple:

	SKBounceAnimation *bounceAnimation = [SKBounceAnimation animationWithKeyPath:@"position.y"];
	bounceAnimation.fromValue = [NSNumber numberWithFloat:view.center.x];
	bounceAnimation.toValue = [NSNumber numberWithFloat:300];
	bounceAnimation.duration = 0.5f;
	bounceAnimation.delegate = self;
	bounceAnimation.numberOfBounces = 2;

	bounceAnimation.removedOnCompletion = NO;
	bounceAnimation.fillMode = kCAFillModeForwards;

	[view.layer addAnimation:bounceAnimation forKey:@"someKey"];

`SKBounceAnimation` is an **explicit** animation, so you have to tell it not to remove upon completion and set the `fillMode` to `kCAFillModeForwards`. Then, in the completion delegate callback, you can set the value of the property to the final value and remove the animation.

	- (void) animationDidStop:(SKBounceAnimation *)animation finished:(BOOL)flag {
		[bouncingView.layer setValue:animation.toValue forKeyPath:animation.keyPath];
		[bouncingView.layer removeAnimationForKey:@"someKey"];
	}

## Math

The math is simple. Check out the [blogpost](http://khanlou.com/2012/01/cakeyframeanimation-make-it-bounce/) and [the informational post](http://khanlou.com/2012/01/dampers-and-their-role-in-physical-models/) preceding it for exact details, but essentially the system behaves with oscillating exponential decay in the form of the equation: `x = Ae^(-αt)•cos(ωt) + B`.

A is the difference between start and end values, B is the end value, α is determined by the number of frames required to get the exponential decay portion to close enough to 0, and ω is determined by the number of periods required to get the desired number of bounces.

## Extras

`shouldOvershoot` is a property that you can change. It defaults to YES; if you set it to NO, the animation will bounce as if it were hitting a wall, instead of overshooting the target value and bouncing back. It looks a lot like the Anvil effect in Keynote.

I’d like to add another property that would make the object stretch away from its original location and then bounce back, the way the photo button in iOS 5.1+ works, but I haven’t had a chance to implement yet.

## Demo app

The demo app contains demos for several different animations that are supported by `SKBounceAnimation`.

* One-axis animation: Using a keypath like `position.x`, we can animate along one axis.
* Two-axis animation: Using a keypath like `position`, `SKBounceAnimation` will generate a path, and your layer will follow it.
* Size: Using the `bounds` keypath, we can make the size increase. The center of the size increase is determined by `anchorPoint`, which can be moved. It defaults to the center of the layer
* Color: I have no idea why anyone would want to bounce a color animation, but I was feeling whimsical, so I added support for this as well.
* Scale: Using a `CATransform3D` struct and the `transform` keypath, we can scale objects. This is very useful to create an effect like UIAlerts bouncing in.
* Scale & Rotate: Using multiple CATransform3Ds on top of each other, we can do super weird effects like scale and rotating. They look really cool.
* Rect: The last demo creates two `SKBounceAnimations` with two different `keyPath`s (`position` and `bounds`) but attaches them to the same layer. The effect looks like a `frame` animation.

## Future work

`SKBounceAnimation` doesn’t support the `byValue` property yet. I also would like to add a property like `stretchAndBounceBack`. 