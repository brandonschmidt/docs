# oref1 (super advanced features)

NOTE OF CAUTION:
oref1 is different than oref0, the baseline "traditional" OpenAPS implementation that only uses temporary basal rates.

## Only run oref1 with the following caveats in mind: 

* Remember that you are choosing to test a still-in-development future. Do so at your own risk & with due diligence to keep yourself safe.
* You should have run oref0 for more than two weeks, and be very aware of all the types of situations in which your rig might fail
* You should have basals of > 0.5 U/hr. (SMB is *not* advisable for those with very small basals; since .1U is the smallest increment that can be bolused by SMB.  We are also adding a basal check to diable SMB when basals are < 0.3 U/hr.)
* Read the following:
  * 1. The updated reference design (https://openaps.org/reference-design/) that explains the differences between oref0 and oref1
  * 2. The following two posts for background on oref1:
  https://diyps.org/2017/04/30/introducing-oref1-and-super-microboluses-smb-and-what-it-means-compared-to-oref0-the-original-openaps-algorithm/
  AND
  https://diyps.org/2017/05/08/choose-one-what-would-you-give-up-if-you-could-with-openaps-maybe-you-can-oref1-includes-unannounced-meals-or-uam/
* Make sure you understand what SMB & UAM stand for (**read the above posts, we know you skipped them**!)
* Plan to have a learning curve. You will interact with oref1 differently when on SMB and UAM than how you were interacting with oref0. In particular: **do not do correction boluses**; use temp targets to give the rig a "nudge". You are very likely to overshoot if you try to do things manually on top of what SMB has already done!

## Understanding SMB

SMB, like all things in OpenAPS, is designed with safety in mind. (Did you skip reading the updated reference design? Go read that first!) SMB is designed to give you reasonably SAFE amounts of bolus needed upfront and use reduced temporary basal rates to safely balance out the peak insulin timing. You are likely to see many long low or zero temps (upwards of 120 minutes long) with SMB turned on, while oref1 is administering SMBs or waiting until it's safe to do so. 

Single SMB amounts are limited by several factors.  The largest a single SMB bolus can be is the SMALLEST value of:

* 30 minutes of the current regular basal rate, or
* 1/3 of the Insulin Required amount, or
* the remaining portion of your maxIOB setting in preferences

## Undertanding UAM 

UAM will be triggered if the preference is toggled on and there is carb activity detected based on positive deviations. 


## How to turn on SMB/UAM

1. As of May 16, 2017, SMB/UAM and other oref1 features are in the dev branch of OpenAPS. You should be comfortable updating your rig to the dev branch.

2. To test SMB, you'll need to run oref0-setup with a "microbolus" enable flag to configure it to use SMB in oref0-pump-loop, and also enable the appropriate enableSMB settings in preferences.json. I would enable them one at a time, in your preferred order, and then closely observe the behavior (via both Nightscout and pump-loop.log) in the enabled situation. In addition to testing it in "normal" situations, pay special attention to how it behaves in more extreme situations, such as with rescue carbs (announced or not), post-meal activity, etc.

**(If you cannot read the code to figure out how to interpret the first sentence in number 2; you may not be ready for SMB.)** 

3. Another optional feature that might help your SMB (or AMA) deal with entered meal carbs is to set an appropriate "remainingCarbsCap". That should be set to no larger than a typical large meal, and no larger than the number of carbs you'd typically be able to absorb over 4 hours. There is a hard-coded maximum of 90 grams, so don't bother setting anything higher than that. If remainingCarbsCap is set to something above zero, then in situations where carbs have been entered but carb absorption has not yet begun, oref1 will assume that 2/3 of those carbs will be absorbed evenly over the next 4 hours. As a result, oref1 can begin SMB'ing (or AMA high-temping) more so than it otherwise would immediately after entering carbs. If carbs are entered early, this can also act more aggressively than "eating soon", based on trusting that you really will eat (most of) the entered carbs at some point before all the insulin kicks in. 

In other words: If it is set to zero, the SMB will be "less aggressive" than if it is set to a non-zero number approximating a large meal. A non-zero number allows the algorithm to deliver insulin earlier even if not seeing a rise in the CGM value. The higher the value the more aggressive it will be. 

4. To test UAM, you'll need to enableUAM in preferences.json. UAM can be enabled without SMB, but it won't be very effective without enableSMB_with_bolus. You'll probably also want to upgrade your Nightscout to a version that includes nightscout/cgm-remote-monitor#2564 To test UAM, you'll first want to be familiar with SMB's "normal" behavior, and then test, with a small meal, giving an up-front bolus (of more than 30m worth of basal, so it can be distinguished from an SMB) and not entering carbs. You'll probably also want to do an "eating soon mode" temporary target or bolus beforehand. You can then observe, by watching the NS purple line predictions and the pump-loop.log, whether your OpenAPS rig is SMB'ing appropriately when it starts to see UAM carb impact.

## Troubleshooting

1. Make sure you read the above, especially the "only enable oref1 if..." section. SMB will behave differently than oref0 would. Watch carefully, and use your common sense and do what's right for you & your diabetes. 
2. Common errors include:
* Not including the enable flag and just changing preferences. You should see "Starting supermicrobolus pump-loop at..." in pump-loop.log if you have successfully enabled everything.
* Pump clock being >1 minute off. This means 60 seconds. Not 61 seconds; 68 seconds; 90 seconds. Needs to be less than 60 seconds apart. `"Checking pump clock: "2017-05-16T15:46:32-04:00" is within 1m of current time: Tue May 16 15:47:40 EDT 2017` is an example of a >60 second gap that needs fixing before it will work properly. We are adding code to automatically attempt to fix this, but until that is merged you'll need to do so manually.
