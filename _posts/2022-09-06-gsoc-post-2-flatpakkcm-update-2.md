---
layout: post
title:  "GSoC Post 2: FlatpakKCM Update 2"
date:   2022-09-06 04:38:37 +0530
categories: jekyll update
---

My previous post in this series tracked what I had done until the 5th week, and gave some information on the technical aspects of the project. This post covers the work done since.

# Work Done:

## 1. Adding User-defined Permissions

For permissions of 4 categories (namely, "Filesystems", "Session Bus Policy", "System Bus Policy" and "Environment"), the user can add their own permissions and an associated value (such as adding their music directory with "read-only", which lets the application read the contents of their music directory).

The first task in the 2nd half of GSoC was adding support for users defining their own paths, buses and environment variables.

At first, I went with keeping a global "Add permission" button, but it was sensibly suggested later on to have a separate "Add New" button for each category.

## 2. Implementing "Apply", "Default" and "Reset" Buttons

The KCM didn't actually work like a KCM because changing a permission on the interface would instantly change the permission in the overrides file as well, instead of sending it to a "waiting" area until the user hits "Apply" button. Similarly, the "Default" and "Reset" buttons did nothing.

Most KCMs use a KConfig file, instead of an overrides file like being used here, to store the settings. This caused me to stall for a while since I wasn't sure how to proceed, but after my mentors referred me to the tablets KCM, work picked up again and I proceeded to implementing the 3 buttons.

This involved overrides 5 functions from the KQuickAddons::ManagedConfigModule class, namely: load(), save(), isDefaults(), isSaveNeeded() and defaults(). This also marked the first time I used signals and slots.

After implementing functionality for the buttons, the next step in this metatask was to handle the situation when the user changes certain permissions, but without clicking any of the buttons that would finalize or reverse the changes, clicks on another application. In the beginning, I did some changes that would do as follows: when the user clicks on an app A and makes some changes and then -- without applying/resetting them -- jumps to another app B and makes some changes there, and then clicks "apply", changes made to B would be applied and changes made to A would still be un-applied, so the user can jump back to A and apply those changes as well. Essentially the 3 buttons worked only for the app and changes that is currently selected.

This was, however, not a good way of doing things. It would be much simpler for a user to just be prevented from changing the application when there are unsaved changes in the presently selected one. So I implemented this instead.

Along the way, as newer changes were made, the "reset" (especially) and the "default" buttons would stop working as expected, so I'd have to revisit this part of the project and fix whatever got broken. These were trivial and not time consuming, though.

## 3. Redesign:

The plan since the beginning was to split the interface into "basic"/"advanced" sections. The basic section would include internet access, device access, remote login, filesystems, bluetooth etc. The advanced section would include more technical things such as interprocess communication, buses and environment variables.

I started working on this a couple of weeks back. During the first week, I split the interface into 3 parts: the app names, option whether the user wants to go to basic permissions or advanced ones, and the permissions themselves. Later, this was changed back to just 2 sections with a drop-down list. When the user clicks it, advanced permissions are shown (or un-shown, if they were displayed already).

Here's how it looks now:

<img src="{{ "/assets/img/redesign_notexp.png" | prepend: site.baseurl | prepend: site.url}}" alt="Screenshot"/>

<img src="{{ "/assets/img/redesign_exp.png" | prepend: site.baseurl | prepend: site.url}}" alt="Screenshot"/>

The 3 above were "metatasks" that involved plenty of minor things. Below, I have mentioned some of the standalone or "minor" things I spent doing:

**1. Fixing section headers' places:** Earlier, section headers would appear as the program read it in metadata or overrides file. For example, if an app does not have any permissions under the "Session Bus Policy" group, Session Bus Policy header would not show up at all. Then, if the user added a session bus, it would show up at the end of the list. The list headers were thus never consistent in their ordering across different applications. So I added them in fixed places using "Dummy" permissions. "Dummy" permissions are just placeholder permissions and their only purpose is to make QML display the section headers. If a permission under a group does not exist, the program would add a dummy permission in that group, so the section appears. Later when the user adds their own permission in that group, the dummy would get deleted. When user deletes their permission, the dummy would come back to life. Dummy permissions are, of course, never shown.

**2. Fixing random crashes:** On random runs, the program would crash. I initially thought it had something to do with the manner in which I was access section labels, but it actually was related to a undefined value being passed to the ref property. The fix was trivial, though it took some time to realize the actual problem.

**3. Fixing icons' problem:** For KDevelop on my own machine, and multiple apps on my mentor's machine, icons would not appear. This took some time to fix since I was not sure what was causing the issue, particularly since the same icons did appear for me on my own machine (except KDevelop).

**4. Adding tooltips, icons etc:** The "add new permission" button needs tooltips and icons, so I added those. I also changed the buttons from raised buttons to flat ones.

**5. Fix bugs related to previous work:** My new changes would break the apply/default/reset buttons from time to time, so I spent a little time fixing these again. Similarly, as I used more and more of the KCM, I realized that in some cases the values for non-simple permissions would not work. However, there has been no problem in the latter part of the project in the past 5 weeks.

**6. Minor design changes:** Preventing components from overflowing, ensuring margins exist in dialogs, making dialog text more specific, improving descriptions of filesystem permissions, putting a frame around the panel, making sure the header on the right view does not get cut off at startup (yet to be merged as of this writing) etc.

**7. Misc. tasks:** Ensuring re-use compliancy and adding licenses properly, addressing warnings from KCM and GTK, cleaning up json file and adding icon etc.

**8. CheckableListItem:** This caused me a fair bit of frustration: the checkable list item delegate would not work if you click the box (instead of elsewhere on the delegate). Using "actions:" or "onCheckedChanged:" instead of "onClicked:" would cause weird behaviour. This was very confusing since the delegate seemed to be working OK in Discover. The fix should be merged soon.

# What's next:

I have fallen of my schedule a considerable amount: I started the Snap KCM part of my project only this weekend. I have listed the app names and icons so far, and I hope to do as much as possible before GSoC ends. I will elaborate upon it in the next post 4-6 days later.
