# Remotely accessible functionality

By default, no functionality is accessible remotely.

When connected locally, a user can opt-in to "Allow remote access" for all or some of a number of groups of actions.

Allowing remote access for a group of actions can be done temporarily (setting an end-date), or permanently.

An admin user can also set this for another user, on their behalf.

An admin user can also see which user currently has remote access to which group of actions.

Once the user has activated remote access for a given group of actions in that way, they can later connect remotely,
and do the same things they would be able to do when inside the house/building.

Example groups of actions:

* user administration (activating new users, granting permissions to users)
* viewing web cam streams and listening to microphone stream
* switching devices on and off, controlling them

# Rationale

The rationale for this restriction is to prevent situations in which some users have access to things that happen inside the
house/building, where other users of the house/building might consider this access a breach of their privacy. For
instance: two people have a conversation in the house, and a third person who is not at home at the time, is listening in
on the conversation.
