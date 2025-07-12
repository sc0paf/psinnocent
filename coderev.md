# Analyzing the Code Review
I assert that the code review video released by Coding Jesus [here]( https://www.youtube.com/watch?v=HHwhiz0s2x8&t=1s) was intentionally misrepresenting PirateSoftware and his positions to gain clout. I typically would have no interest in creating a document like this - but it has become clear that this is just the most recent shot fired by a cancel-mob who refuse to acknowledge any nuance in the SKG proposals and assertions. I think its worth briefly mentioning the larger strokes of that issue and its misrepresentation involving PirateSoftware before we continue.

## SKG Concerns & Opposal
**The Strawman**: "*PirateSoftware* thinks that people shouldn't have a right to own their games and supports shady developers rights to rip off their consumers and revoke access whenever they want.""

**The Actual Concerns**: 
	*Stop Killing Games (SKG)* as is would put an unfair burden on devs if forced through as-is, and the crowd behind SKG is completely unwilling to acknowledge these exceptions or modify their demands. Those are:

1. Live service games rely on a lot of assets and services that they simply cannot give away. They don't own them, they don't own the servers or the services that run them, and are simply NOT in a position to do so.
2. Subscription based games like MMOs have never even pretended to have the server software on offer. This is entirely seperate software and IP in general that the end-user has no right to demand. You buy a client and a subscription to use the server. Consumers at large know this to be true, I think, but SKG wishes to demand that software anyway. 
3. Generally speaking- games designed around online multi-player at scale were not designed to include a single-player experience. Removing support for the game universally indicates that the game is no-longer financially viable, so forcing that publisher to dumb a bunch of resources into a game that is no longer viable is unteneble, and an unfair burden to place on publishers.

On the flip side: I can not imagine that PirateSoftware- or ANY consumer would argue against SKG's take on the portion of the market that targets games that explicitly single-player, and the removal of support is things like losing access to a day-1 patch, or losing access to DRM.

# My Assertions Regarding Review
This code review was intentionally disingenuous and was *only* attempting to cast PirateSoftware in the worst light possible. There are several recurring themes that point to this:

1. **Total lack of consulting the GML documentation** - You don't have to learn the entire language to review it- but you should probably at least make sure you're not critizing the naming of a function that is actually an internal method of the engine. The same can be said about those functions parameters, obviously. 

2. **Referencing best practices as if they are rules** - Best practices are best pratices. You can criticize someone for not following them, but if you're reviewing code that nobody asked you to review and you crucify them for not worshipping at the altar of clean code- you are just being a dick. Its important to note for those who lack any programming knowledge that "clean code" is something pushed like a religion by a lot of those who endorse it. That isn't to say that it isn't good practice- but it isn't *mandatory* by any means. It compiles to the same shit in the end. This is a *very* common way that programmers attempt to snub their nose at other programmers and is something akin to your local grammar-nazi.

## The Opening
Coding Jesus opens by asserting his qualifications. It is entirely disingenuous for a C++ developer to pretend like theres no difference between a dynamically typed scripting language that works as an extension for Game Maker and a strictly typed language like C++. It is made very clear that the video is a hit-piece and not a genuine review. 

## The first bit of Code
The entirety of the first portion of reviewed code is criticisms of the engine itself, not the code on screen. You would expect that a developer with a ton of experience would note the IDE hinting at this. What I mean is:


```ts
// Falling Particle
effect_drop = part_type_create();
part_type_sprite(effect_drop, spr_spark, false, false, true);
```

What you see above is the very first piece of code we see on screen being criticized. Very quickly googling "GML part_type_create" or "GML part_type_sprite" immediately reveals that these are internal GML methods. Further- the implication that single method calls creating a throwaway effect shouldn't have "magic numbers" is absurd. It is a commonly reccommended best practice- but you will *constantly* find people not doing this, because there is a trade-off between needless abstraction and readability. For example:

Lets say we have an internal method that is exposed to us that creates a square. The documentation tells you:
```
create_square(width, height, filled, color)

Width (number) The width of the square
Height (number) The height of the square
color: (Color) The color of the square
filled (boolean) Should the square be filled with color
```

In more inconsequential portions of the code, or honestly even largely, many developers would find this entirely acceptable:
```
const stupidSquare = create_square(300, 300, true, 'blue');
```

The assertion here is that these "magic numbers" are unacceptable, but theres nothing inherently problematic about this - especially if you have a quick comment explaining it. If you are someone who is familiar with this method and you're the only one working on it, its perfectly reasonable to not document it at all. You may be familiar to the degree that explanation is pointless because nobody else is going to work on it, and you know it well. The assertion is that these should be more descriptive:
```
const square_width = 300
const square_height = 300
const square_filled = true
const square_color = 'blue'
const stupidSquare = create_square(square_width, square_height, square_filled, square_color)
```

What you see above is perfectly valid, of course. The argument in favor of it is that it makes your code more maintainable, for others and for your future self. However, there are perfectly valid arguments why this is unnecessary abstraction. The trade-off is "time and effort now" vs "time and effort later." You would have to demonstrate that *not* doing this actually caused any issues here to support the claim that NOT abstracting out every parameter was problematic. Modern IDEs will have hover hints or alt hints or popups that will contain the parameter info that you need 9 times out 10. Even if they don't, maybe this is a simple method and you are familiar with the parameters. The process hes not describing is not free- and for an individual developer the up-front time cost might just not be worth it to them. I we can take the first bit of code shown to demonstrate how this might be true:

```
// Falling Particle
effect_drop = part_type_create();
part_type_sprite(effect_drop, spr_spark, false, false, true)
part_type_size(effect_drop, 1, 1, 1, 0);
part_type_life(effect_drop, 15, 15);
part_type_alpha1(effect_drop, 1);
```

becomes:
```
effect_drop = part_type_create();
const sprite = spr_spark;
const isAnimated = false;
const isStretched = false;
const isRandomSubImage = true;

part_type_sprite(effect_drop, sprite, isAnimated, isStretched, isRandomSubImage);

const sizeMin = 1;
const sizeMax = 1;
const sizeStep = 1;
const wiggleAmount = 0;
part_type_size(effect_drop, sizeMin, sizeMax, sizeStep, wiggleAmount);

const particleLifeMin = 15;
const particleLifeMax = 15;
part_type_life(effect_drop, particleLifeMin, particleLifeMax);

// .. etc
```
And that pattern would continue for each subsequent function call. Does it really seem so unreasonable to *not* balloon your code to that degree? Those who bend the knee to clean code purists would may say "but you must!!!" but *plenty of people* would say this is perfectly reasonable.

The secondary solution they offered here was equally disingenuous. The claim that "this is how you must do it" isn't really valid in a dynamically typed language like GML. **GML is dyanamically (and very weakly) typed.** There is no compiler-enforced type checking. Type compatability would only be determined at runtime and error on the function call- not at compile time. As such, you wouldn't *inherently expect* someone to reach for that type of pattern using this language. In fact, this was such an annoying problem for people in Javascript that they invented Typescript. Beyond that- with the `part-type-*` being internal methods to the engine, the signature *is also baked into the engine.* It expects those same 5 arguments of those types, every time. You can't redefine them. You can't overload them. The best you could do is create a wrapper function:
```
/// @func setup_drop_particle(pp, cfg)
/// @arg pp ParticleType
/// @arg cfg Struct with keys sprite, animate, stretch, random
function setup_drop_particle(pp, cfg) {
  part_type_sprite(pp, cfg.sprite, cfg.animate, cfg.stretch, cfg.random);
  // …etc…
}
```

...but if you're going to create wrapper functions for all the internal methods exposed to you, why are you even using the engine? This is, again, applying standards of C++ to GML that don't really make sense. 

One final note about these methods: The GML documentation itself shows the functions being used this way- including passing the booleans as numbers. Here are the examples from `part_type_sprite`:

```
global.p1 = part_type_create();
part_type_sprite(global.p1 , spr_Coins, 1, 0, 0);
part_type_size(global.p1, 1, 3, 0, 0);
part_type_scale(global.p1, 1, 1);
part_type_colour1(global.p1, c_white);
part_type_alpha2(global.p1, 1, 0);
part_type_speed(global.p1, 2, 4, 0, 0);
part_type_direction(global.p1, 0, 180, 0, 0);
part_type_gravity(global.p1, 0.20, 270);
part_type_orientation(global.p1, 0, 0, 0, 0, 1);
part_type_blend(global.p1, 1);
part_type_life(global.p1, 15, 60);
```

Its unreasonable to export someone to appeal to some standard *other than the engine's own documentation*.

## The Second Bit of Code

### Alarms
This is equally pedantic. 

`alarm[0]` `alarm[1]` `alarm[2]` : These are again built into each object. From the GML docs, again:

>This 1-dimensional array is used to get the current value for any alarms that the instance may have, or it can be used to set those alarms. There are twelve alarms built into each instance of an object, and each one has its own event that will run when this variable reaches 0.

I'm not going to harp on this because I think it has covered, but they're built into the objects as an array of these alarms. They take integers. They're used numerically even in the case of on/off because they contain a countdown. It wouldn't make very much sense to have them run by decrementing a value and then ending that chain as false. They use a -1 check rather than a null check, but again the difference there would be strictly pedantic. 


### The Conditionals & Comments
This might have been the only valid criticism I saw up until this point in the video. GML provides constants of 'true' and 'false' according to their documentation that do actually just represent 1 and 0, so while the code isn't necessarily *wrong*, I don't see why you wouldn't just use the constants. In looking through GML history, I did find that GML pretty recently got several modern features that are standard in most languages- so its possible there was some weird quirks when this code was written? That said, even though I don't have a defense for this, its still just a *bad preference*, and not *broken code.* 

All the stuff about comments is annoyingly at odds with the theme of the rest of the video. This is something that the clean-code purists push pretty hard, but you would think if someone wasn't using all the 'self-documenting code' conventions they harp on, they would see at least having comments as the necessary runner-up. To me, drilling self-documenting code but denouncing over-commenting comes off more as "you should be doing what we say" than "you should do these things that make your code easier to understand."  "Superfluous" and "waste of space" for a comment after demanding that every argument you ever pass to a function be put in a descriptively named variable first? Hello pot, meet kettle.


### The Storyline Array
You can see immediately when this is on screen that the only purpose of any part of this review was to belittle and take part in the bandwagoning and throw stones. The reviewer can be seen barely able to contain their excitement to trash this setup. Game maker doesn't seem to support yaml, theres no parser, no ide packagaing. Your choices are ini or JSON - and I would bet that array *is* dumped to JSON to save the game (but I can't be sure of this). Also- I don't think think the reviewer understands the concept of objectivity. You can't say "This is objectively better because it makes it harder to make mistakes in the future" - thats not how objectivity works. Claiming something is *objectively better* only makes sense in the context of what you're optimizing for and how you measure it. For example, all of the suggestions made would be "objectively better" if your objective if "conforming in the strictest sense to all clean code conventions to the best of my ability" - then sure. "Objective" !== "My Preference writ large." If you finish the whole project without running into problems and saved time by not using a more verbose system, then no amount error-proofing state management would have been "objectively better." If you have a system that is working for you that you are comfortable with, introducing a different system because it has benefits that don't matter to you is not "objectively better." 

There is again, though, nothing about this setup that doesn't work. Without seeing the full context of the code, it hard to tell one way or the other whether this is just a flat out bad approach or not. There are more reasons why this wouldn't be a good approach than there are reasons it would be a good approach, but that doesn't *negate the fact that the former could exist.* For example, PS made mention of an ARG that utilizes the code as gameplay mechanics. Maybe this ended up being the best middle ground for maintaining the puzzles aspect of the ARG while also being seriveable state management for the game? How could you possibly know this isn't the case? We are, again, deep into territory of "my preferences === good code."

Toward the end of this segment, there is mention made of it being "really really weird that hes using these numbers to represent people ..." This is another assumption based on 3 lines of code. How could you even claim to definitively know that this represents a person, by the number? This could just as easily be a switch representing possible groupings based on that specific state. E.g.:
```ts
switch(global.storyline_array[419]) {
	case 1:
		// This involves Shelly and Doug
	case 2:
		// this involves Craig and Steve
	case 3:
		// This just involves the toaster
}
```

You couldn't know that. This is another assumption. "Imagine removing all these comments" - he states. Why is this even a scenario worth imagining? Why would anyone do that? "Imagine if we changed the language to French. Would you have any idea what any of this dialog says?" No- I wouldn't. But thats also a stupid hypothetical.

## A Final & Meta Note
You can see throughout the bits of code shown that these are scripts attached to a visual editor. I haven't used the visual editor either- but I think its insane to assume that you would have unlimited freedom in something like this to rewrite all their methods and change their classes and just generally have universal freedom to re-write how everything works to better support your preferred coding paradigm (which by the way, isn't necessarily going to match others). I guess we shouldn't be surprised that someone who calls himself "Jesus" maintains a holier-than-thou attitude about his preferred coding conventions- but make no mistake. This is a video that asserts a bunch of preferences as rules and refers to a bunch of features from C++ that don't even make sense in GML. 

You can be mad about Thor's takes on SKG. That doesn't mean you have to jump on the bandwagon of clout-chasing elitist "muh best practices" jackasses who want to join in on tearing people down to help boost their own fame. Try contending with the arguments directly rather than trying to assassinate the source's character. Its not a good look- and you just look like a jackass trying to throw unapplicable industry terms around.
