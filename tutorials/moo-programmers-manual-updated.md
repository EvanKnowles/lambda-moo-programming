# LambdaMOO Programmer's Manual

## For LambdaMOO Version 1.8.*

Originally written <time datetime="1997-03-01 16:00">March, 1997</time>:

<address>by Pavel Curtis, aka Haakon, aka Lambda</address>

Updated <time datetime="2018-07-19 17:00">July, 2018</time> ([CHANGE LOG](CHANGELOG.md)):

<address>By [Brendan Butts](http://github.com/sevenecks), aka Slither, aka Fengshui</address>

For older versions of this document please see the resources section.

## LambdaMOO Resources

*   [Lambda MOO Programming Resources GitHub](https://github.com/SevenEcks/lambda-moo-programming)
*   [lisdude MOO resources](http://www.lisdude.com/moo/)
*   [Unedited MOO Programmers Manual](http://www.hayseed.net/MOO/manuals/ProgrammersManual.html)
*   [Older Unedited MOO Programmers Mnaual](http://www2.iath.virginia.edu/courses/moo/ProgrammersManual.texinfo_toc.html)
*   [LambdaMOO Source (github)](https://github.com/SevenEcks/LambdaMOO)
*   [LambdaMOO Databases (and other resources)](http://lambda.moo.mud.org/pub/MOO/)
*   [MOO Talk Mailing List](https://groups.google.com/forum/#!forum/MOO-talk)
*   [Dome Client Web Socket MOO Client](https://github.com/JavaChilly/dome-client.js)

## Table of Contents

*   [Introduction](#introduction)

* * *

## Foreword

Hey, it's me: Brendan. The guy updating this document. The idea of updating the MOO Programming Manual has crossed my mind a few times. It's a bit of a daunting task. This document is pretty comprehensive and I'm endeavouring to make it even more so. I'm adding some much needed styling so that it looks more modern. Things were once hosted primarily on FTP servers and we now have GitHub, so many links and resources mentioned in this document have been updated and I've even added some additional ones.

If you want an up to date manual on MOO, this is the guide for you. If you want to read the historic document which contains the unedited text, I recommend reviewing the links toward the top of the page that point to unedited versions of the guide.

## Introduction

LambdaMOO is a network-accessible, multi-user, programmable, interactive system well-suited to the construction of text-based adventure games, conferencing systems, and other collaborative software. Its most common use, however, is as a multi-participant, low-bandwidth virtual reality, and it is with this focus in mind that I describe it here.

Participants (usually referred to as _players_) connect to LambdaMOO using Telnet or some other, more specialized, _client_ program. Upon connection, they are usually presented with a _welcome message_ explaining how to either create a new _character_ or connect to an existing one. Characters are the embodiment of players in the virtual reality that is LambdaMOO.

<div class="well"><span class="label label-info">Note:</span> No one really connects to a MOO over telnet these days. MOO Clients and MUD Clients are relatively common. See the resources section for more information on these. There are even some web based clients (dome-client) out there that use websockets to connect to a MOO directly from the browser.</div>

Having connected to a character, players then give one-line commands that are parsed and interpreted by LambdaMOO as appropriate. Such commands may cause changes in the virtual reality, such as the location of a character, or may simply report on the current state of that reality, such as the appearance of some object.

The job of interpreting those commands is shared between the two major components in the LambdaMOO system: the _server_ and the _database_. The server is a program, written in a standard programming language, that manages the network connections, maintains queues of commands and other tasks to be executed, controls all access to the database, and executes other programs written in the MOO programming language. The database contains representations of all the objects in the virtual reality, including the MOO programs that the server executes to give those objects their specific behaviors.

Almost every command is parsed by the server into a call on a MOO procedure, or _verb_, that actually does the work. Thus, programming in the MOO language is a central part of making non-trivial extensions to the database and thus, the virtual reality.

In the next chapter, I describe the structure and contents of a LambdaMOO database. The following chapter gives a complete description of how the server performs its primary duty: parsing the commands typed by players. Next, I describe the complete syntax and semantics of the MOO programming language. Finally, I describe all of the database conventions assumed by the server.

<div class="well"><span class="label label-info">Note:</span> For the most part, this manual describes only those aspects of LambdaMOO that are entirely independent of the contents of the database. It does not describe, for example, the commands or programming interfaces present in the LambdaCore database. There are exceptions to this, for situations where it seems prudent to delve deeper into these areas.</div>

## The LambdaMOO Database

In this chapter, I begin by describing in detail the various kinds of data that can appear in a LambdaMOO database and that, therefore, MOO programs can manipulate. In a few places, I refer to the _LambdaCore_ database. This is one particular LambdaMOO database, created every so often by extracting the "core" of the current database for the original LambdaMOO.

<div class="well"><span class="label label-info">Note:</span> The original LambdaMOO resides on the host <span class="cmd">lambda.parc.xerox.com</span> (the numeric address for which is <span class="cmd">192.216.54.2</span>), on port 8888\. Feel free to drop by! A copy of the most recent release of the LambdaCore database can be obtained by anonymous FTP from host <span class="cmd">ftp.parc.xerox.com</span> in the directory <span class="cmd">pub/MOO</span>.</div>

<div class="alert alert-info">The above information may be out of date, but the most recent dump of the _LambdaCore_ can be found [here](http://lambda.moo.mud.org/pub/MOO/).</div>

### MOO Value Types

There are only a few kinds of values that MOO programs can manipulate:

*   integers (in a specific, large range)
*   real numbers (represented with floating-point numbers)
*   strings (of characters)
*   objects (in the virtual reality)
*   errors (arising during program execution)
*   lists (of all of the above, including lists)

MOO supports the integers from -2^31 (that is, negative two to the power of 31) up to 2^31 - 1 (one less than two to the power of 31); that's from -2147483648 to 2147483647, enough for most purposes. In MOO programs, integers are written just as you see them here, an optional minus sign followed by a non-empty sequence of decimal digits. In particular, you may not put commas, periods, or spaces in the middle of large integers, as we sometimes do in English and other natural languages (e.g., `2,147,483,647').

Real numbers in MOO are represented as they are in almost all other programming languages, using so-called _floating-point_ numbers. These have certain (large) limits on size and precision that make them useful for a wide range of applications. Floating-point numbers are written with an optional minus sign followed by a non-empty sequence of digits punctuated at some point with a decimal point (`.') and/or followed by a scientific-notation marker (the letter `E' or `e' followed by an optional sign and one or more digits). Here are some examples of floating-point numbers:

<pre>325.0   325\.  3.25e2   0.325E3   325.E1   .0325e+4   32500e-2</pre>

All of these examples mean the same number. The third of these, as an example of scientific notation, should be read "3.25 times 10 to the power of 2".

<div class="well"><span class="label label-info">Fine point:</span> The MOO represents floating-point numbers using the local meaning of the C-language <span class="cmd">double</span> type, which is almost always equivalent to IEEE 754 double precision floating point. If so, then the smallest positive floating-point number is no larger than <span class="cmd">2.2250738585072014e-308</span> and the largest floating-point number is <span class="cmd">1.7976931348623157e+308</span>.

*   IEEE infinities and NaN values are not allowed in MOO.
*   The error <span class="cmd">E_FLOAT</span> is raised whenever an infinity would otherwise be computed.
*   The error <span class="cmd">E_INVARG</span> is raised whenever a NaN would otherwise arise.
*   The value <span class="cmd">0.0</span> is always returned on underflow.

</div>

Character _strings_ are arbitrarily-long sequences of normal, ASCII printing characters. When written as values in a program, strings are enclosed in double-quotes, like this:

<pre>"This is a character string."</pre>

To include a double-quote in the string, precede it with a backslash (<span class="cmd">\</span>), like this:

<pre>"His name was \"Leroy\", but nobody ever called him that."</pre>

Finally, to include a backslash in a string, double it:

<pre>"Some people use backslash ('\\') to mean set difference."</pre>

MOO strings may not include special ASCII characters like carriage-return, line-feed, bell, etc. The only non-printing characters allowed are spaces and tabs.

<div class="well"><span class="label label-info">Fine point:</span> There is a special kind of string used for representing the arbitrary bytes used in general, binary input and output. In a _binary string_, any byte that isn't an ASCII printing character or the space character is represented as the three-character substring "~XX", where XX is the hexadecimal representation of the byte; the input character `~' is represented by the three-character substring "~7E". This special representation is used by the functions <span class="cmd">encode_binary()</span> and <span class="cmd">decode_binary()</span> and by the functions <span class="cmd">notify()</span> and <span class="cmd">read()</span> with network connections that are in binary mode. See the descriptions of the <span class="cmd">set_connection_option()</span>, <span class="cmd">encode_binary()</span>, and <span class="cmd">decode_binary()</span> functions for more details.</div>

_Objects_ are the backbone of the MOO database and, as such, deserve a great deal of discussion; the entire next section is devoted to them. For now, let it suffice to say that every object has a number, unique to that object.

In programs, we write a reference to a particular object by putting a hash mark (<span class="cmd">#</span>) followed by the number, like this:

<pre>#495</pre>

<div class="well"><span class="label label-info">Note:</span> Referencing object numbers in your code should be discouraged. An object only exists until it is recycled. It is technically possible for an object number to change under some circumstances. Thus, you should use a corified reference to an object ($my_special_object) instead. More on corified references later.</div>

Object numbers are always integers.

There are three special object numbers used for a variety of purposes: <span class="cmd">#-1</span>, <span class="cmd">#-2</span>, and <span class="cmd">#-3</span>, usually referred to in the LambdaCore database as <span class="cmd">$nothing</span>, <span class="cmd">$ambiguous_match</span>, and <span class="cmd">$failed_match</span>, respectively.

_Errors_ are, by far, the least frequently used values in MOO. In the normal case, when a program attempts an operation that is erroneous for some reason (for example, trying to add a number to a character string), the server stops running the program and prints out an error message. However, it is possible for a program to stipulate that such errors should not stop execution; instead, the server should just let the value of the operation be an error value. The program can then test for such a result and take some appropriate kind of recovery action. In programs, error values are written as words beginning with <span class="cmd">E_</span>. The complete list of error values, along with their associated messages, is as follows:

<dl class="dl-horizontal">

<dt>E_NONE</dt>

<dd>No error</dd>

<dt>E_TYPE</dt>

<dd>Type mismatch</dd>

<dt>E_DIV</dt>

<dd>Division by zero</dd>

<dt>E_PERM</dt>

<dd>Permission denied</dd>

<dt>E_PROPNF</dt>

<dd>Property not found</dd>

<dt>E_VERBNF</dt>

<dd>Verb not found</dd>

<dt>E_VARNF</dt>

<dd>Variable not found</dd>

<dt>E_INVIND</dt>

<dd>Invalid indirection</dd>

<dt>E_RECMOVE</dt>

<dd>Recursive move</dd>

<dt>E_MAXREC</dt>

<dd>Too many verb calls</dd>

<dt>E_RANGE</dt>

<dd>Range error</dd>

<dt>E_ARGS</dt>

<dd>Incorrect number of arguments</dd>

<dt>E_NACC</dt>

<dd>Move refused by destination</dd>

<dt>E_INVARG</dt>

<dd>Invalid argument</dd>

<dt>E_QUOTA</dt>

<dd>Resource limit exceeded</dd>

<dt>E_FLOAT</dt>

<dd>Floating-point arithmetic error</dd>

</dl>

<div class="well"><span class="label label-info">Note:</span> The MOO can be extended using extensions. These are plugins that usually contain patch files that are applied directly to your LambdaMOO source code. One very popular extension is the FileIO plugin, which allows you to read and write files to disk. Extensions can throw their own errors, though it's not as common. Old versions of FileIO threw a fake error "E_FILE", though a recently updated version throws true E_FILE errors.</div>

The final kind of value in MOO programs is _lists_. A list is a sequence of arbitrary MOO values, possibly including other lists. In programs, lists are written in mathematical set notation with each of the elements written out in order, separated by commas, the whole enclosed in curly braces (<span class="cmd">{</span> and <span class="cmd">}</span>). For example, a list of the names of the days of the week is written like this:

<pre>{"Sunday", "Monday", "Tuesday", "Wednesday",
 "Thursday", "Friday", "Saturday"}
</pre>

<div class="well"><span class="label label-info">Note:</span> It doesn't matter that we put a line-break in the middle of the list. This is true in general in MOO: anywhere that a space can go, a line-break can go, with the same meaning. The only exception is inside character strings, where line-breaks are not allowed.</div>

### Objects in the MOO Database

Objects are, in a sense, the whole point of the MOO programming language. They are used to represent objects in the virtual reality, like people, rooms, exits, and other concrete things. Because of this, MOO makes a bigger deal out of creating objects than it does for other kinds of value, like integers.

Numbers always exist, in a sense; you have only to write them down in order to operate on them. With objects, it is different. The object with number <span class="cmd">#958</span> does not exist just because you write down its number. An explicit operation, the <span class="cmd">create()</span> function described later, is required to bring an object into existence. Symmetrically, once created, objects continue to exist until they are explicitly destroyed by the <span class="cmd">recycle()</span> function (also described later).

The identifying number associated with an object is unique to that object. It was assigned when the object was created and will never be reused, even if the object is destroyed. Thus, if we create an object and it is assigned the number <span class="cmd">#1076</span>, the next object to be created will be assigned <span class="cmd">#1077</span>, even if <span class="cmd">#1076</span> is destroyed in the meantime.

<div class="alert alert-info">The above limitation led to design of systems to manage object reuse. The <span class="cmd">$recycler</span> is one example of such a system. This is **not** present in the <span class="cmd">minimal.db</span> which is included in the LambdaMOO source, however it is present in the latest dump of the [LambdaCore DB](http://lambda.moo.mud.org/pub/MOO/) which is the recommended starting point for new development.</div>

Every object is made up of three kinds of pieces that together define its behavior: _attributes_, _properties_, and _verbs_.

#### Fundamental Object Attributes

There are three fundamental _attributes_ to every object:

1.  A flag (either true or false) specifying whether or not the object represents a player
2.  The object that is its _parent_
3.  A list of the objects that are its _children_; that is, those objects for which this object is their parent.

The act of creating a character sets the player attribute of an object and only a wizard (using the function <span class="cmd">set_player_flag()</span>) can change that setting. Only characters have the player bit set to 1.

The parent/child hierarchy is used for classifying objects into general classes and then sharing behavior among all members of that class. For example, the LambdaCore database contains an object representing a sort of "generic" room. All other rooms are _descendants_ (i.e., children or children's children, or ...) of that one. The generic room defines those pieces of behavior that are common to all rooms; other rooms specialize that behavior for their own purposes. The notion of classes and specialization is the very essence of what is meant by _object-oriented_ programming. Only the functions <span class="cmd">create()</span>, <span class="cmd">recycle()</span>, <span class="cmd">chparent()</span>, and <span class="cmd">renumber()</span> can change the parent and children attributes.

#### Properties on Objects

A _property_ is a named "slot" in an object that can hold an arbitrary MOO value. Every object has eight built-in properties whose values are constrained to be of particular types. In addition, an object can have any number of other properties, none of which have type constraints. The built-in properties are as follows:

<dl class="dl-horizontal">

<dt>name</dt>

<dd>a string, the usual name for this object</dd>

<dt>owner</dt>

<dd>an object, the player who controls access to it</dd>

<dt>location</dt>

<dd>an object, where the object is in virtual reality</dd>

<dt>contents</dt>

<dd>a list of objects, the inverse of <span class="cmd">location</span></dd>

<dt>programmer</dt>

<dd>a bit, does the object have programmer rights?</dd>

<dt>wizard</dt>

<dd>a bit, does the object have wizard rights?</dd>

<dt>r</dt>

<dd>a bit, is the object publicly readable?</dd>

<dt>w</dt>

<dd>a bit, is the object publicly writable?</dd>

<dt>f</dt>

<dd>a bit, is the object fertile?</dd>

</dl>

The <span class="cmd">name</span> property is used to identify the object in various printed messages. It can only be set by a wizard or by the owner of the object. For player objects, the <span class="cmd">name</span> property can only be set by a wizard; this allows the wizards, for example, to check that no two players have the same name.

The <span class="cmd">owner</span> identifies the object that has owner rights to this object, allowing them, for example, to change the <span class="cmd">name</span> property. Only a wizard can change the value of this property.

The <span class="cmd">location</span> and <span class="cmd">contents</span> properties describe a hierarchy of object containment in the virtual reality. Most objects are located "inside" some other object and that other object is the value of the <span class="cmd">location</span> property.

The <span class="cmd">contents</span> property is a list of those objects for which this object is their location. In order to maintain the consistency of these properties, only the <span class="cmd">move()</span> function is able to change them.

The <span class="cmd">wizard</span> and <span class="cmd">programmer</span> bits are only applicable to characters, objects representing players. They control permission to use certain facilities in the server. They may only be set by a wizard.

The <span class="cmd">r</span> bit controls whether or not players other than the owner of this object can obtain a list of the properties or verbs in the object.

Symmetrically, the <span class="cmd">w</span> bit controls whether or not non-owners can add or delete properties and/or verbs on this object. The <span class="cmd">r</span> and <span class="cmd">w</span> bits can only be set by a wizard or by the owner of the object.

The <span class="cmd">f</span> bit specifies whether or not this object is _fertile_, whether or not players other than the owner of this object can create new objects with this one as the parent. It also controls whether or not non-owners can use the <span class="cmd">chparent()</span> built-in function to make this object the parent of an existing object. The <span class="cmd">f</span> bit can only be set by a wizard or by the owner of the object.

All of the built-in properties on any object can, by default, be read by any player. It is possible, however, to override this behavior from within the database, making any of these properties readable only by wizards. See the chapter on server assumptions about the database for details.

As mentioned above, it is possible, and very useful, for objects to have other properties aside from the built-in ones. These can come from two sources.

First, an object has a property corresponding to every property in its parent object. To use the jargon of object-oriented programming, this is a kind of _inheritance_. If some object has a property named <span class="cmd">foo</span>, then so will all of its children and thus its children's children, and so on.

Second, an object may have a new property defined only on itself and its descendants. For example, an object representing a rock might have properties indicating its weight, chemical composition, and/or pointiness, depending upon the uses to which the rock was to be put in the virtual reality.

Every defined property (as opposed to those that are built-in) has an owner and a set of permissions for non-owners. The owner of the property can get and set the property's value and can change the non-owner permissions. Only a wizard can change the owner of a property.

The initial owner of a property is the player who added it; this is usually, but not always, the player who owns the object to which the property was added. This is because properties can only be added by the object owner or a wizard, unless the object is publicly writable (i.e., its <span class="cmd">w</span> property is 1), which is rare. Thus, the owner of an object may not necessarily be the owner of every (or even any) property on that object.

The permissions on properties are drawn from this set: <span class="cmd">r</span> (read), <span class="cmd">w</span> (write), and <span class="cmd">c</span> (change ownership in descendants). Read permission lets non-owners get the value of the property and, of course, write permission lets them set that value. The <span class="cmd">c</span> permission bit is a little more complicated.

Recall that every object has all of the properties that its parent does and perhaps some more. Ordinarily, when a child object inherits a property from its parent, the owner of the child becomes the owner of that property. This is because the <span class="cmd">c</span> permission bit is "on" by default. If the <span class="cmd">c</span> bit is not on, then the inherited property has the same owner in the child as it does in the parent.

As an example of where this can be useful, the LambdaCore database ensures that every player has a <span class="cmd">password</span> property containing the encrypted version of the player's connection password. For security reasons, we don't want other players to be able to see even the encrypted version of the password, so we turn off the <span class="cmd">r</span> permission bit. To ensure that the password is only set in a consistent way (i.e., to the encrypted version of a player's password), we don't want to let anyone but a wizard change the property. Thus, in the parent object for all players, we made a wizard the owner of the password property and set the permissions to the empty string, <span class="cmd">""</span>. That is, non-owners cannot read or write the property and, because the <span class="cmd">c</span> bit is not set, the wizard who owns the property on the parent class also owns it on all of the descendants of that class.

<div class="alert alert-warning">**Warning:** The MOO will only hash the first 8 characters of a password. In practice this means that the passwords <span class="cmd">password</span> and <span class="cmd">password12345</span> are exactly the same and either one can be used to login.</div>

Another, perhaps more down-to-earth example arose when a character named Ford started building objects he called "radios" and another character, yduJ, wanted to own one. Ford kindly made the generic radio object fertile, allowing yduJ to create a child object of it, her own radio. Radios had a property called <span class="cmd">channel</span> that identified something corresponding to the frequency to which the radio was tuned. Ford had written nice programs on radios (verbs, discussed below) for turning the channel selector on the front of the radio, which would make a corresponding change in the value of the <span class="cmd">channel</span> property. However, whenever anyone tried to turn the channel selector on yduJ's radio, they got a permissions error. The problem concerned the ownership of the <span class="cmd">channel</span> property.

As I explain later, programs run with the permissions of their author. So, in this case, Ford's nice verb for setting the channel ran with his permissions. But, since the <span class="cmd">channel</span> property in the generic radio had the <span class="cmd">c</span> permission bit set, the <span class="cmd">channel</span> property on yduJ's radio was owned by her. Ford didn't have permission to change it! The fix was simple. Ford changed the permissions on the <span class="cmd">channel</span> property of the generic radio to be just <span class="cmd">r</span>, without the <span class="cmd">c</span> bit, and yduJ made a new radio. This time, when yduJ's radio inherited the <span class="cmd">channel</span> property, yduJ did not inherit ownership of it; Ford remained the owner. Now the radio worked properly, because Ford's verb had permission to change the channel.

#### Verbs on Objects

The final kind of piece making up an object is _verbs_. A verb is a named MOO program that is associated with a particular object. Most verbs implement commands that a player might type; for example, in the LambdaCore database, there is a verb on all objects representing containers that implements commands of the form <span class="cmd">put <var>object</var> in <var>container</var></span>.

It is also possible for MOO programs to invoke the verbs defined on objects. Some verbs, in fact, are designed to be used only from inside MOO code; they do not correspond to any particular player command at all. Thus, verbs in MOO are like the _procedures_ or _methods_ found in some other programming languages.

<div class="alert alert-info"><span class="label label-info">Note:</span> There are even more ways to refer to _verbs_ and their counterparts in other programming language: _procedure_, _function_, _subroutine_, _subprogram_, and _method_ are the primary onces. However, in _Object Oriented Programming_ abbreviated _OOP_ you may primarily know them as methods.</div>

As with properties, every verb has an owner and a set of permission bits. The owner of a verb can change its program, its permission bits, and its argument specifiers (discussed below). Only a wizard can change the owner of a verb.

The owner of a verb also determines the permissions with which that verb runs; that is, the program in a verb can do whatever operations the owner of that verb is allowed to do and no others. Thus, for example, a verb owned by a wizard must be written very carefully, since wizards are allowed to do just about anything.

<div class="alert alert-warning"><span class="label label-warning">Warning:</span> This is serious business. The MOO has a variety of checks in place for permissions (at the object, verb and property levels) that are all but ignored when a verb is executing with a wizard's permisisons. You may want to create a non-wizard character and give them the programmer bit, and write much of your code there, leaving the wizard bit for things that actually require access to everything, despite permissions.</div>

The permission bits on verbs are drawn from this set: <span class="cmd">r</span> (read), <span class="cmd">w</span> (write), <span class="cmd">x</span> (execute), and <span class="cmd">d</span> (debug). Read permission lets non-owners see the program for a verb and, symmetrically, write permission lets them change that program. The other two bits are not, properly speaking, permission bits at all; they have a universal effect, covering both the owner and non-owners.

The execute bit determines whether or not the verb can be invoked from within a MOO program (as opposed to from the command line, like the <span class="cmd">put</span> verb on containers). If the <span class="cmd">x</span> bit is not set, the verb cannot be called from inside a program. The <span class="cmd">x</span> bit is usually set.

The setting of the debug bit determines what happens when the verb's program does something erroneous, like subtracting a number from a character string. If the <span class="cmd">d</span> bit is set, then the server _raises_ an error value; such raised errors can be _caught_ by certain other pieces of MOO code. If the error is not caught, however, the server aborts execution of the command and, by default, prints an error message on the terminal of the player whose command is being executed. (See the chapter on server assumptions about the database for details on how uncaught errors are handled.) If the <span class="cmd">d</span> bit is not set, then no error is raised, no message is printed, and the command is not aborted; instead the error value is returned as the result of the erroneous operation.

<div class="well"><span class="label label-info">Note:</span> The <span class="cmd">d</span> bit exists only for historical reasons; it used to be the only way for MOO code to catch and handle errors. With the introduction of the <span class="cmd">try</span>-<span class="cmd">except</span> statement and the error-catching expression, the <span class="cmd">d</span> bit is no longer useful. All new verbs should have the <span class="cmd">d</span> bit set, using the newer facilities for error handling if desired. Over time, old verbs written assuming the <span class="cmd">d</span> bit would not be set should be changed to use the new facilities instead.</div>

In addition to an owner and some permission bits, every verb has three _argument specifiers_, one each for the <span class="cmd">direct object</span>, the <span class="cmd">preposition</span>, and the <span class="cmd">indirect object</span>. The direct and indirect specifiers are each drawn from this set: <span class="cmd">this</span>, <span class="cmd">any</span>, or <span class="cmd">none</span>. The preposition specifier is <span class="cmd">none</span>, <span class="cmd">any</span>, or one of the items in this list:

*   with/using
*   at/to
*   in front of
*   in/inside/into
*   on top of/on/onto/upon
*   out of/from inside/from
*   over
*   through
*   under/underneath/beneath
*   behind
*   beside
*   for/about
*   is
*   as
*   off/off of

The argument specifiers are used in the process of parsing commands, described in the next chapter.

## The Built-in Command Parser

The MOO server is able to do a small amount of parsing on the commands that a player enters. In particular, it can break apart commands that follow one of the following forms:

*   <var>verb</var>
*   <var>verb</var> <var>direct-object</var>
*   <var>verb</var> <var>direct-object</var> <var>preposition</var> <var>indirect-object</var>

Real examples of these forms, meaningful in the LambdaCore database, are as follows:

<pre>look
take yellow bird
put yellow bird in cuckoo clock
</pre>

Note that English articles (i.e., <span class="cmd">the</span>, <span class="cmd">a</span>, and <span class="cmd">an</span>) are not generally used in MOO commands; the parser does not know that they are not important parts of objects' names.

To have any of this make real sense, it is important to understand precisely how the server decides what to do when a player types a command.

First, the server checks whether or not the first non-blank character in the command is one of the following:

*   <span class="cmd"><var>"</var></span>
*   <span class="cmd"><var>:</var></span>
*   <span class="cmd"><var>;</var></span>

If so, that character is replaced by the corresponding command below, followed by a space:

*   <span class="cmd"><var>say</var></span>
*   <span class="cmd"><var>emote</var></span>
*   <span class="cmd"><var>eval</var></span>

For example this command:

<div class="alert"><span class="cmd">"Hi, there.</span></div>

will be treated exactly as if it were as follows:

<div class="alert"><span class="cmd">say Hi, there.</span></div>

The server next breaks up the command into words. In the simplest case, the command is broken into words at every run of space characters; for example, the command <span class="cmd">foo bar baz</span> would be broken into the words <span class="cmd">foo</span>, <span class="cmd">bar</span>, and <span class="cmd">baz</span>. To force the server to include spaces in a "word", all or part of a word can be enclosed in double-quotes. For example, the command:

<div class="alert"><span class="cmd">foo "bar mumble" baz" "fr"otz" bl"o"rt</span></div>

is broken into the words <span class="cmd">foo</span>, <span class="cmd">bar mumble</span>, <span class="cmd">baz frotz</span>, and <span class="cmd">blort</span>.

Finally, to include a double-quote or a backslash in a word, they can be preceded by a backslash, just like in MOO strings.

Having thus broken the string into words, the server next checks to see if the first word names any of the six "built-in" commands:

*   <span class="cmd">.program</span>
*   <span class="cmd">PREFIX</span>
*   <span class="cmd">OUTPUTPREFIX</span>
*   <span class="cmd">SUFFIX</span>
*   <span class="cmd">OUTPUTSUFFIX</span>
*   or the connection's defined _flush_ command, if any (<span class="cmd">.flush</span> by default).

The first one of these is only available to programmers, the next four are intended for use by client programs, and the last can vary from database to database or even connection to connection; all six are described in the final chapter of this document, "Server Commands and Database Assumptions". If the first word isn't one of the above, then we get to the usual case: a normal MOO command.

The server next gives code in the database a chance to handle the command. If the verb <span class="cmd">$do_command()</span> exists, it is called with the words of the command passed as its arguments and <span class="cmd">argstr</span> set to the raw command typed by the user. If <span class="cmd">$do_command()</span> does not exist, or if that verb-call completes normally (i.e., without suspending or aborting) and returns a false value, then the built-in command parser is invoked to handle the command as described below. Otherwise, it is assumed that the database code handled the command completely and no further action is taken by the server for that command.

If the built-in command parser is invoked, the server tries to parse the command into a verb, direct object, preposition and indirect object. The first word is taken to be the verb. The server then tries to find one of the prepositional phrases listed at the end of the previous section, using the match that occurs earliest in the command. For example, in the very odd command <span class="cmd">foo as bar to baz</span>, the server would take <span class="cmd">as</span> as the preposition, not <span class="cmd">to</span>.

If the server succeeds in finding a preposition, it considers the words between the verb and the preposition to be the direct object and those after the preposition to be the indirect object. In both cases, the sequence of words is turned into a string by putting one space between each pair of words. Thus, in the odd command from the previous paragraph, there are no words in the direct object (i.e., it is considered to be the empty string, <span class="cmd">""</span>) and the indirect object is <span class="cmd">"bar to baz"</span>.

If there was no preposition, then the direct object is taken to be all of the words after the verb and the indirect object is the empty string.

The next step is to try to find MOO objects that are named by the direct and indirect object strings.

First, if an object string is empty, then the corresponding object is the special object <span class="cmd">#-1</span> (aka <span class="cmd">$nothing</span> in LambdaCore). If an object string has the form of an object number (i.e., a hash mark (<span class="cmd">#</span>) followed by digits), and the object with that number exists, then that is the named object. If the object string is either <span class="cmd">"me"</span> or <span class="cmd">"here"</span>, then the player object itself or its location is used, respectively.

<div class="alert alert-info">

<span class="label label-info">Note:</span> $nothing is considered a <span class="cmd">corified</span> object. This means that a _property_ has been created on <span class="cmd">#0</span> named <span class="cmd">nothing</span> with the value of <span class="cmd">#-1</span>. For example (after creating the property): <span class="cmd">;#0.nothing = #-1</span>

This allows you to reference the <span class="cmd">#-1</span> object via it's corified reference of <span class="cmd">$nothing</span>. In practice this can be very useful as you can use corified references in your code (and should!) instead of object numbers.

Among other benefits this allows you to write your code (which references other objects) once and then swap out the corified reference, pointing to a different object.

For instance if you have a new errror logging system and you want to replace the old $error_logger reference with your new one, you wont have to find all the references to the old error logger object number in your code. You can just change the property on <span class="cmd">#0</span> to reference the new object.

</div>

Otherwise, the server considers all of the objects whose location is either the player (i.e., the objects the player is "holding", so to speak) or the room the player is in (i.e., the objects in the same room as the player); it will try to match the object string against the various names for these objects.

The matching done by the server uses the <span class="cmd">aliases</span> property of each of the objects it considers. The value of this property should be a list of strings, the various alternatives for naming the object. If it is not a list, or the object does not have an <span class="cmd">aliases</span> property, then the empty list is used. In any case, the value of the <span class="cmd">name</span> property is added to the list for the purposes of matching.

The server checks to see if the object string in the command is either exactly equal to or a prefix of any alias; if there are any exact matches, the prefix matches are ignored. If exactly one of the objects being considered has a matching alias, that object is used. If more than one has a match, then the special object <span class="cmd">#-2</span> (aka <span class="cmd">$ambiguous_match</span> in LambdaCore) is used. If there are no matches, then the special object <span class="cmd">#-3</span> (aka <span class="cmd">$failed_match</span> in LambdaCore) is used.

So, now the server has identified a verb string, a preposition string, and direct- and indirect-object strings and objects. It then looks at each of the verbs defined on each of the following four objects, in order:

1.  the player who typed the command
2.  the room the player is in
3.  the direct object, if any
4.  the indirect object, if any.

For each of these verbs in turn, it tests if all of the the following are true:

*   the verb string in the command matches one of the names for the verb
*   the direct- and indirect-object values found by matching are allowed by the corresponding _argument specifiers_ for the verb
*   the preposition string in the command is matched by the _preposition specifier_ for the verb.

I'll explain each of these criteria in turn.

Every verb has one or more names; all of the names are kept in a single string, separated by spaces. In the simplest case, a verb-name is just a word made up of any characters other than spaces and stars (i.e., ` ' and <span class="cmd">*</span>). In this case, the verb-name matches only itself; that is, the name must be matched exactly.

If the name contains a single star, however, then the name matches any prefix of itself that is at least as long as the part before the star. For example, the verb-name <span class="cmd">foo*bar</span> matches any of the strings <span class="cmd">foo</span>, <span class="cmd">foob</span>, <span class="cmd">fooba</span>, or <span class="cmd">foobar</span>; note that the star itself is not considered part of the name.

If the verb name _ends_ in a star, then it matches any string that begins with the part before the star. For example, the verb-name <span class="cmd">foo*</span> matches any of the strings <span class="cmd">foo</span>, <span class="cmd">foobar</span>, <span class="cmd">food</span>, or <span class="cmd">foogleman</span>, among many others. As a special case, if the verb-name is <span class="cmd">*</span> (i.e., a single star all by itself), then it matches anything at all.

Recall that the argument specifiers for the direct and indirect objects are drawn from the set <span class="cmd">none</span>, <span class="cmd">any</span>, and <span class="cmd">this</span>. If the specifier is <span class="cmd">none</span>, then the corresponding object value must be <span class="cmd">#-1</span> (aka <span class="cmd">$nothing</span> in LambdaCore); that is, it must not have been specified. If the specifier is <span class="cmd">any</span>, then the corresponding object value may be anything at all. Finally, if the specifier is <span class="cmd">this</span>, then the corresponding object value must be the same as the object on which we found this verb; for example, if we are considering verbs on the player, then the object value must be the player object.

Finally, recall that the argument specifier for the preposition is either <span class="cmd">none</span>, <span class="cmd">any</span>, or one of several sets of prepositional phrases, given above. A specifier of <span class="cmd">none</span> matches only if there was no preposition found in the command. A specifier of <span class="cmd">any</span> always matches, regardless of what preposition was found, if any. If the specifier is a set of prepositional phrases, then the one found must be in that set for the specifier to match.

So, the server considers several objects in turn, checking each of their verbs in turn, looking for the first one that meets all of the criteria just explained. If it finds one, then that is the verb whose program will be executed for this command. If not, then it looks for a verb named <span class="cmd">huh</span> on the room that the player is in; if one is found, then that verb will be called. This feature is useful for implementing room-specific command parsing or error recovery. If the server can't even find a <span class="cmd">huh</span> verb to run, it prints an error message like <span class="cmd">I couldn't understand that.</span> and the command is considered complete.

At long last, we have a program to run in response to the command typed by the player. When the code for the program begins execution, the following built-in variables will have the indicated values:

<dl class="dl-horizontal">

<dt>player</dt>

<dd>an object, the player who typed the command</dd>

<dt>this</dt>

<dd>an object, the object on which this verb was found</dd>

<dt>caller</dt>

<dd>an object, the same as <span class="cmd">player</span></dd>

<dt>verb</dt>

<dd>a string, the first word of the command</dd>

<dt>argstr</dt>

<dd>a string, everything after the first word of the command</dd>

<dt>args</dt>

<dd>a list of strings, the words in <span class="cmd">argstr</span></dd>

<dt>dobjstr</dt>

<dd>a string, the direct object string found during parsing</dd>

<dt>dobj</dt>

<dd>an object, the direct object value found during matching</dd>

<dt>prepstr</dt>

<dd>a string, the prepositional phrase found during parsing</dd>

<dt>iobjstr</dt>

<dd>a string, the indirect object string</dd>

<dt>iobj</dt>

<dd>an object, the indirect object value</dd>

</dl>

The value returned by the program, if any, is ignored by the server.

## The MOO Programming Language

MOO stands for "MUD, Object Oriented." MUD, in turn, has been said to stand for many different things, but I tend to think of it as "Multi-User Dungeon" in the spirit of those ancient precursors to MUDs, Adventure and Zork.

MOO, the programming language, is a relatively small and simple object-oriented language designed to be easy to learn for most non-programmers; most complex systems still require some significant programming ability to accomplish, however.

Having given you enough context to allow you to understand exactly what MOO code is doing, I now explain what MOO code looks like and what it means. I begin with the syntax and semantics of expressions, those pieces of code that have values. After that, I cover statements, the next level of structure up from expressions. Next, I discuss the concept of a task, the kind of running process initiated by players entering commands, among other causes. Finally, I list all of the built-in functions available to MOO code and describe what they do.

First, though, let me mention comments. You can include bits of text in your MOO program that are ignored by the server. The idea is to allow you to put in notes to yourself and others about what the code is doing. To do this, begin the text of the comment with the two characters <span class="cmd">/*</span> and end it with the two characters <span class="cmd">*/</span>; this is just like comments in the C programming language. Note that the server will completely ignore that text; it will _not_ be saved in the database. Thus, such comments are only useful in files of code that you maintain outside the database.

To include a more persistent comment in your code, try using a character string literal as a statement. For example, the sentence about peanut butter in the following code is essentially ignored during execution but will be maintained in the database:

<pre>for x in (players())
  "Grendel eats peanut butter!";
  player:tell(x.name, " (", x, ")");
endfor
</pre>

<div class="alert alert-info"><span class="label label-info">Note:</span> In practice, the only style of comments you will use is quoted strings of text. Get used to it.</div>

### MOO Language Expressions

Expressions are those pieces of MOO code that generate values; for example, the MOO code

<pre>3 + 4
</pre>

is an expression that generates (or "has" or "returns") the value 7. There are many kinds of expressions in MOO, all of them discussed below.

#### Errors While Evaluating Expressions

Most kinds of expressions can, under some circumstances, cause an error to be generated. For example, the expression <span class="cmd">x / y</span> will generate the error <span class="cmd">E_DIV</span> if <span class="cmd">y</span> is equal to zero. When an expression generates an error, the behavior of the server is controlled by setting of the <span class="cmd">d</span> (debug) bit on the verb containing that expression. If the <span class="cmd">d</span> bit is not set, then the error is effectively squelched immediately upon generation; the error value is simply returned as the value of the expression that generated it.

<div class="well"><span class="label label-info">Note:</span> This error-squelching behavior is very error prone, since it affects _all_ errors, including ones the programmer may not have anticipated. The <span class="cmd">d</span> bit exists only for historical reasons; it was once the only way for MOO programmers to catch and handle errors. The error-catching expression and the <span class="cmd">try</span>-<span class="cmd">except</span> statement, both described below, are far better ways of accomplishing the same thing.</div>

If the <span class="cmd">d</span> bit is set, as it usually is, then the error is _raised_ and can be caught and handled either by code surrounding the expression in question or by verbs higher up on the chain of calls leading to the current verb. If the error is not caught, then the server aborts the entire task and, by default, prints a message to the current player. See the descriptions of the error-catching expression and the <span class="cmd">try</span>-<span class="cmd">except</span> statement for the details of how errors can be caught, and the chapter on server assumptions about the database for details on the handling of uncaught errors.

#### Writing Values Directly in Verbs

The simplest kind of expression is a literal MOO value, just as described in the section on values at the beginning of this document. For example, the following are all expressions:

*   17
*   #893
*   "This is a character string."
*   E_TYPE
*   {"This", "is", "a", "list", "of", "words"}

In the case of lists, like the last example above, note that the list expression contains other expressions, several character strings in this case. In general, those expressions can be of any kind at all, not necessarily literal values. For example,

<pre>{3 + 4, 3 - 4, 3 * 4}
</pre>

is an expression whose value is the list <span class="cmd">{7, -1, 12}</span>.

#### Naming Values Within a Verb

As discussed earlier, it is possible to store values in properties on objects; the properties will keep those values forever, or until another value is explicitly put there. Quite often, though, it is useful to have a place to put a value for just a little while. MOO provides local variables for this purpose.

Variables are named places to hold values; you can get and set the value in a given variable as many times as you like. Variables are temporary, though; they only last while a particular verb is running; after it finishes, all of the variables given values there cease to exist and the values are forgotten.

Variables are also "local" to a particular verb; every verb has its own set of them. Thus, the variables set in one verb are not visible to the code of other verbs.

The name for a variable is made up entirely of letters, digits, and the underscore character (<span class="cmd">_</span>) and does not begin with a digit. The following are all valid variable names:

*   foo
*   _foo
*   this2that
*   M68000
*   two_words
*   This_is_a_very_long_multiword_variable_name

Note that, along with almost everything else in MOO, the case of the letters in variable names is insignificant. For example, these are all names for the same variable:

*   fubar
*   Fubar
*   FUBAR
*   fUbAr

A variable name is itself an expression; its value is the value of the named variable. When a verb begins, almost no variables have values yet; if you try to use the value of a variable that doesn't have one, the error value <span class="cmd">E_VARNF</span> is raised. (MOO is unlike many other programming languages in which one must _declare_ each variable before using it; MOO has no such declarations.) The following variables always have values:

*   INT
*   FLOAT
*   OBJ
*   STR
*   LIST
*   ERR
*   player
*   this
*   caller
*   verb
*   args
*   argstr
*   dobj
*   dobjstr
*   prepstr
*   iobj
*   iobjstr
*   NUM

The values of some of these variables always start out the same:

<dl class="dl-horizontal">

<dt><span class="cmd">INT</span></dt>

<dd>an integer, the type code for integers (see the description of the function <span class="cmd">typeof()</span>, below)</dd>

<dt><span class="cmd">NUM</span></dt>

<dd>the same as <span class="cmd">INT</span> (for historical reasons)</dd>

<dt><span class="cmd">FLOAT</span></dt>

<dd>an integer, the type code for floating-point numbers</dd>

<dt><span class="cmd">LIST</span></dt>

<dd>an integer, the type code for lists</dd>

<dt><span class="cmd">STR</span></dt>

<dd>an integer, the type code for strings</dd>

<dt><span class="cmd">OBJ</span></dt>

<dd>an integer, the type code for objects</dd>

<dt><span class="cmd">ERR</span></dt>

<dd>an integer, the type code for error values</dd>

</dl>

For others, the general meaning of the value is consistent, though the value itself is different for different situations:

<dl class="dl-horizontal">

<dt><span class="cmd">player</span></dt>

<dd>an object, the player who typed the command that started the task that involved running this piece of code.</dd>

<dt><span class="cmd">this</span></dt>

<dd>an object, the object on which the currently-running verb was found.</dd>

<dt><span class="cmd">caller</span></dt>

<dd>an object, the object on which the verb that called the currently-running verb was found. For the first verb called for a given command, <span class="cmd">caller</span> has the same value as <span class="cmd">player</span>.</dd>

<dt><span class="cmd">verb</span></dt>

<dd>a string, the name by which the currently-running verb was identified.</dd>

<dt><span class="cmd">args</span></dt>

<dd>a list, the arguments given to this verb. For the first verb called for a given command, this is a list of strings, the words on the command line.</dd>

</dl>

The rest of the so-called "built-in" variables are only really meaningful for the first verb called for a given command. Their semantics is given in the discussion of command parsing, above.

To change what value is stored in a variable, use an _assignment_ expression:

<pre><var>variable</var> = <var>expression</var>
</pre>

For example, to change the variable named <span class="cmd">x</span> to have the value 17, you would write <span class="cmd">x = 17</span> as an expression. An assignment expression does two things:

*   it changes the value of of the named variable
*   it returns the new value of that variable

Thus, the expression

<pre>13 + (x = 17)
</pre>

changes the value of <span class="cmd">x</span> to be 17 and returns 30.

#### Arithmetic Operators

All of the usual simple operations on numbers are available to MOO programs:

*   +
*   -
*   *
*   /
*   %

These are, in order, addition, subtraction, multiplication, division, and remainder. In the following table, the expressions on the left have the corresponding values on the right:

<pre>5 + 2       =>   7
5 - 2       =>   3
5 * 2       =>   10
5 / 2       =>   2
5.0 / 2.0   =>   2.5
5 % 2       =>   1
5.0 % 2.0   =>   1.0
5 % -2      =>   1
-5 % 2      =>   -1
-5 % -2     =>   -1
-(5 + 2)    =>   -7
</pre>

Note that integer division in MOO throws away the remainder and that the result of the remainder operator (<span class="cmd">%</span>) has the same sign as the left-hand operand. Also, note that <span class="cmd">-</span> can be used without a left-hand operand to negate a numeric expression.

<div class="well"><span class="label label-info">Fine point:</span> Integers and floating-point numbers cannot be mixed in any particular use of these arithmetic operators; unlike some other programming languages, MOO does not automatically coerce integers into floating-point numbers. You can use the <span class="cmd">tofloat()</span> function to perform an explicit conversion.</div>

The <span class="cmd">+</span> operator can also be used to append two strings. The expression <span class="cmd">"foo" + "bar"</span> has the value <span class="cmd">"foobar"</span>

Unless both operands to an arithmetic operator are numbers of the same kind (or, for <span class="cmd">+</span>, both strings), the error value <span class="cmd">E_TYPE</span> is raised. If the right-hand operand for the division or remainder operators (<span class="cmd">/</span> or <span class="cmd">%</span>) is zero, the error value <span class="cmd">E_DIV</span> is raised.

MOO also supports the exponentiation operation, also known as "raising to a power," using the <span class="cmd">^</span> operator:

<pre>3 ^ 4       =>   81
3 ^ 4.5     error-->   E_TYPE
3.5 ^ 4     =>   150.0625
3.5 ^ 4.5   =>   280.741230801382
</pre>

Note that if the first operand is an integer, then the second operand must also be an integer. If the first operand is a floating-point number, then the second operand can be either kind of number. Although it is legal to raise an integer to a negative power, it is unlikely to be terribly useful.

#### Comparing Values

Any two values can be compared for equality using <span class="cmd">==</span> and <span class="cmd">!=</span>. The first of these returns 1 if the two values are equal and 0 otherwise; the second does the reverse:

<pre>3 == 4                              =>  0
3 != 4                              =>  1
3 == 3.0                            =>  0
"foo" == "Foo"                      =>  1
#34 != #34                          =>  0
{1, #34, "foo"} == {1, #34, "FoO"}  =>  1
E_DIV == E_TYPE                     =>  0
3 != "foo"                          =>  1
</pre>

Note that integers and floating-point numbers are never equal to one another, even in the _obvious_ cases. Also note that comparison of strings (and list values containing strings) is case-insensitive; that is, it does not distinguish between the upper- and lower-case version of letters. To test two values for case-sensitive equality, use the <span class="cmd">equal</span> function described later.

<div class="alert alert-warning"><span class="label label-warning">Warning:</span> It is easy (and very annoying) to confuse the equality-testing operator (<span class="cmd">==</span>) with the assignment operator (<span class="cmd">=</span>), leading to nasty, hard-to-find bugs. Don't do this.</div>

Numbers, object numbers, strings, and error values can also be compared for ordering purposes using the following operators:

<dl class="dl-horizontal">

<dt><</dt>

<dd>meaning "less than"</dd>

<dt><=</dt>

<dd>"less than or equal"</dd>

<dt>>=</dt>

<dd>"greater than or equal"</dd>

<dt>></dt>

<dd>"greater than"</dd>

</dl>

As with the equality operators, these return 1 when their operands are in the appropriate relation and 0 otherwise:

<pre>3 < 4           =>  1
3 < 4.0         =>  E_TYPE (an error)
#34 >= #32      =>  1
"foo" <= "Boo"  =>  0
E_DIV > E_TYPE  =>  1
</pre>

Note that, as with the equality operators, strings are compared case-insensitively. To perform a case-sensitive string comparison, use the <span class="cmd">strcmp</span> function described later. Also note that the error values are ordered as given in the table in the section on values. If the operands to these four comparison operators are of different types (even integers and floating-point numbers are considered different types), or if they are lists, then <span class="cmd">E_TYPE</span> is raised.

#### Values as True and False

There is a notion in MOO of _true_ and _false_ values; every value is one or the other. The true values are as follows:

*   all integers other than zero
*   all floating-point numbers not equal to <span class="cmd">0.0</span>
*   all non-empty strings (i.e., other than <span class="cmd">""</span>)
*   all non-empty lists (i.e., other than <span class="cmd">{}</span>)

<div class="alert alert-warning"><span class="label label-warning">Warning:</span> Negative numbers are considered true.</div>

All other values are false:

*   the integer zero
*   the floating-point numbers <span class="cmd">0.0</span> and <span class="cmd">-0.0</span>
*   the empty string (<span class="cmd">""</span>)
*   the empty list (<span class="cmd">{}</span>)
*   all object numbers
*   all error values

<div class="alert alert-info"><span class="label label-info">Note:</span> Objects are considered false. If you need to evaluate if a value is of the type object, you can use <span class="cmd">typeof(potential_object) == OBJ</span> however, keep in mind that this does not mean that the object referenced actually exists. IE: #100000000 will return true, but that does not mean that object exists in your MOO.</div>

There are four kinds of expressions and two kinds of statements that depend upon this classification of MOO values. In describing them, I sometimes refer to the _truth value_ of a MOO value; this is just _true_ or _false_, the category into which that MOO value is classified.

The conditional expression in MOO has the following form:

<pre><var>expression-1</var> ? <var>expression-2</var> | <var>expression-3</var>
</pre>

<div class="alert alert-info"><span class="label label-info">Note:</span> This is commonly refered to as a ternary statement in most programming languages.</div>

First, <var>expression-1</var> is evaluated. If it returns a true value, then <var>expression-2</var> is evaluated and whatever it returns is returned as the value of the conditional expression as a whole. If <var>expression-1</var> returns a false value, then <var>expression-3</var> is evaluated instead and its value is used as that of the conditional expression.

<pre>1 ? 2 | 3           =>  2
0 ? 2 | 3           =>  3
"foo" ? 17 | {#34}  =>  17
</pre>

Note that only one of <var>expression-2</var> and <var>expression-3</var> is evaluated, never both.

To negate the truth value of a MOO value, use the <span class="cmd">!</span> operator:

<pre>! <var>expression</var>
</pre>

If the value of <var>expression</var> is true, <span class="cmd">!</span> returns 0; otherwise, it returns 1:

<pre>! "foo"     =>  0
! (3 >= 4)  =>  1
</pre>

The negation operator is usually read as "not."

<div class="alert alert-info"><span class="label label-info">Note:</span> The "negation" or "not" operator is commonly referred to as "bang" by many programmers.</div>

It is frequently useful to test more than one condition to see if some or all of them are true. MOO provides two operators for this:

<pre><var>expression-1</var> && <var>expression-2</var>
<var>expression-1</var> || <var>expression-2</var>
</pre>

These operators are usually read as "and" and "or," respectively.

The <span class="cmd">&&</span> operator first evaluates <var>expression-1</var>. If it returns a true value, then <var>expression-2</var> is evaluated and its value becomes the value of the <span class="cmd">&&</span> expression as a whole; otherwise, the value of <var>expression-1</var> is used as the value of the <span class="cmd">&&</span> expression.

<div class="alert alert-info"><span class="alert alert-info">Note:</span> <var>expression-2</var> is only evaluated if <var>expression-1</var> returns a true value.</div>

The <span class="cmd">&&</span> expression is equivalent to the conditional expression:

<pre><var>expression-1</var> ? <var>expression-2</var> | <var>expression-1</var>
</pre>

except that <var>expression-1</var> is only evaluated once.

The <span class="cmd">||</span> operator works similarly, except that <var>expression-2</var> is evaluated only if <var>expression-1</var> returns a false value. It is equivalent to the conditional expression:

<pre><var>expression-1</var> ? <var>expression-1</var> | <var>expression-2</var>
</pre>

except that, as with <span class="cmd">&&</span>, <var>expression-1</var> is only evaluated once.

These two operators behave very much like "and" and "or" in English:

<pre>1 && 1                  =>  1
0 && 1                  =>  0
0 && 0                  =>  0
1 || 1                  =>  1
0 || 1                  =>  1
0 || 0                  =>  0
17 <= 23  &&  23 <= 27  =>  1
</pre>

#### Indexing into Lists and Strings

Both strings and lists can be seen as ordered sequences of MOO values. In the case of strings, each is a sequence of single-character strings; that is, one can view the string <span class="cmd">"bar"</span> as a sequence of the strings <span class="cmd">"b"</span>, <span class="cmd">"a"</span>, and <span class="cmd">"r"</span>. MOO allows you to refer to the elements of lists and strings by number, by the _index_ of that element in the list or string. The first element in a list or string has index 1, the second has index 2, and so on.

<div class="alert alert-warning"><span class="alert alert-warning">Warning:</span> It is very important to note that unlike many programming languages (which use 0 as the starting index), MOO uses 1.</div>

##### Extracting an Element from a List or String

The indexing expression in MOO extracts a specified element from a list or string:

<pre><var>expression-1</var>[<var>expression-2</var>]
</pre>

First, <var>expression-1</var> is evaluated; it must return a list or a string (the _sequence_). Then, <var>expression-2</var> is evaluated and must return an integer (the _index_). If either of the expressions returns some other type of value, <span class="cmd">E_TYPE</span> is returned. The index must be between 1 and the length of the sequence, inclusive; if it is not, then <span class="cmd">E_RANGE</span> is raised. The value of the indexing expression is the index'th element in the sequence. Anywhere within <var>expression-2</var>, you can use the symbol <span class="cmd">$</span> as an expression returning the length of the value of <var>expression-1</var>.

<pre>"fob"[2]                =>  "o"
"fob"[1]                =>  "f"
{#12, #23, #34}[$ - 1]  =>  #23
</pre>

Note that there are no legal indices for the empty string or list, since there are no integers between 1 and 0 (the length of the empty string or list).

<div class="well"><span class="label label-info">Fine point:</span> The <span class="cmd">$</span> expression actually returns the length of the value of the expression just before the nearest enclosing <span class="cmd">[...]</span> indexing or subranging brackets. For example:

<pre>"frob"[{3, 2, 4}[$]]     =>  "b"
</pre>

is possible because $ in this case represents the 3rd index of the list next to it, which evaluates to the value 4, which in turn is applied as the index to the string, which evaluates to the b.</div>

##### Replacing an Element of a List or String

It often happens that one wants to change just one particular slot of a list or string, which is stored in a variable or a property. This can be done conveniently using an _indexed assignment_ having one of the following forms:

<pre><var>variable</var>[<var>index-expr</var>] = <var>result-expr</var>
<var>object-expr</var>.<var>name</var>[<var>index-expr</var>] = <var>result-expr</var>
<var>object-expr</var>.(<var>name-expr</var>)[<var>index-expr</var>] = <var>result-expr</var>
$<var>name</var>[<var>index-expr</var>] = <var>result-expr</var>
</pre>

The first form writes into a variable, and the last three forms write into a property. The usual errors (<span class="cmd">E_TYPE</span>, <span class="cmd">E_INVIND</span>, <span class="cmd">E_PROPNF</span> and <span class="cmd">E_PERM</span> for lack of read/write permission on the property) may be raised, just as in reading and writing any object property; see the discussion of object property expressions below for details.

Correspondingly, if <var>variable</var> does not yet have a value (i.e., it has never been assigned to), <span class="cmd">E_VARNF</span> will be raised.

If <var>index-expr</var> is not an integer, or if the value of <var>variable</var> or the property is not a list or string, <span class="cmd">E_TYPE</span> is raised. If <var>result-expr</var> is a string, but not of length 1, <span class="cmd">E_INVARG</span> is raised. Now suppose <var>index-expr</var> evaluates to an integer <var>n</var>. If <var>n</var> is outside the range of the list or string (i.e. smaller than 1 or greater than the length of the list or string), <span class="cmd">E_RANGE</span> is raised. Otherwise, the actual assignment takes place.

For lists, the variable or the property is assigned a new list that is identical to the original one except at the <var>n</var>-th position, where the new list contains the result of <var>result-expr</var> instead. For strings, the variable or the property is assigned a new string that is identical to the original one, except the <var>n</var>-th character is changed to be <var>result-expr</var>.

The assignment expression itself returns the value of <var>result-expr</var>. For the following examples, assume that <span class="cmd">l</span> initially contains the list <span class="cmd">{1, 2, 3}</span> and that <span class="cmd">s</span> initially contains the string "foobar":

<pre>l[5] = 3          =>   E_RANGE (error)
l["first"] = 4    =>   E_TYPE  (error)
s[3] = "baz"      =>   E_INVARG (error)
l[2] = l[2] + 3   =>   5
l                 =>   {1, 5, 3}
l[2] = "foo"      =>   "foo"
l                 =>   {1, "foo", 3}
s[2] = "u"        =>   "u"
s                 =>   "fuobar"
s[$] = "z"        =>   "z"
s                 =>   "fuobaz"
</pre>

<div class="alert alert-info"><span class="label label-info">Note:</span> (error) is only used for formatting and identification purposes in these examples and is not present in an actual raised error on the MOO.</div>

<div class="alert alert-info"><span class="label label-info">Note:</span> The <span class="cmd">$</span> expression may also be used in indexed assignments with the same meaning as before.</div>

<div class="well"><span class="label label-info">Fine point:</span> After an indexed assignment, the variable or property contains a _new_ list or string, a copy of the original list in all but the <var>n</var>-th place, where it contains a new value. In programming-language jargon, the original list is not mutated, and there is no aliasing. (Indeed, no MOO value is mutable and no aliasing ever occurs.)</div>

In the list case, indexed assignment can be nested to many levels, to work on nested lists. Assume that <span class="cmd">l</span> initially contains the list:

<pre>{{1, 2, 3}, {4, 5, 6}, "foo"}
</pre>

in the following examples:

<pre>l[7] = 4             =>   E_RANGE (error)
l[1][8] = 35         =>   E_RANGE (error)
l[3][2] = 7          =>   E_TYPE (error)
l[1][1][1] = 3       =>   E_TYPE (error)
l[2][2] = -l[2][2]   =>   -5
l                    =>   {{1, 2, 3}, {4, -5, 6}, "foo"}
l[2] = "bar"         =>   "bar"
l                    =>   {{1, 2, 3}, "bar", "foo"}
l[2][$] = "z"        =>   "z"
l                    =>   {{1, 2, 3}, "baz", "foo"}
</pre>

The first two examples raise <span class="cmd">E_RANGE</span> because 7 is out of the range of <span class="cmd">l</span> and 8 is out of the range of <span class="cmd">l[1]</span>. The next two examples raise <span class="cmd">E_TYPE</span> because <span class="cmd">l[3]</span> and <span class="cmd">l[1][1]</span> are not lists.

##### Extracting a Subsequence of a List or String

The range expression extracts a specified subsequence from a list or string:

<pre><var>expression-1</var>[<var>expression-2</var>..<var>expression-3</var>]
</pre>

The three expressions are evaluated in order. <var>Expression-1</var> must return a list or string (the _sequence_) and the other two expressions must return integers (the _low_ and _high_ indices, respectively); otherwise, <span class="cmd">E_TYPE</span> is raised. The <span class="cmd">$</span> expression can be used in either or both of <var>expression-2</var> and <var>expression-3</var> just as before, meaning the length of the value of <var>expression-1</var>.

If the low index is greater than the high index, then the empty string or list is returned, depending on whether the sequence is a string or a list. Otherwise, both indices must be between 1 and the length of the sequence; <span class="cmd">E_RANGE</span> is raised if they are not. A new list or string is returned that contains just the elements of the sequence with indices between the low and high bounds.

<pre>"foobar"[2..$]                       =>  "oobar"
"foobar"[3..3]                       =>  "o"
"foobar"[17..12]                     =>  ""
{"one", "two", "three"}[$ - 1..$]    =>  {"two", "three"}
{"one", "two", "three"}[3..3]        =>  {"three"}
{"one", "two", "three"}[17..12]      =>  {}
</pre>

##### Replacing a Subsequence of a List or String

The subrange assigment replaces a specified subsequence of a list or string with a supplied subsequence. The allowed forms are:

<pre><var>variable</var>[<var>start-index-expr</var>..<var>end-index-expr</var>] = <var>result-expr</var>
<var>object-expr</var>.<var>name</var>[<var>start-index-expr</var>..<var>end-index-expr</var>] = <var>result-expr</var>
<var>object-expr</var>.(<var>name-expr</var>)[<var>start-index-expr</var>..<var>end-index-expr</var>] = <var>result-expr</var>
$<var>name</var>[<var>start-index-expr</var>..<var>end-index-expr</var>] = <var>result-expr</var>
</pre>

As with indexed assigments, the first form writes into a variable, and the last three forms write into a property. The same errors (<span class="cmd">E_TYPE</span>, <span class="cmd">E_INVIND</span>, <span class="cmd">E_PROPNF</span> and <span class="cmd">E_PERM</span> for lack of read/write permission on the property) may be raised. If <var>variable</var> does not yet have a value (i.e., it has never been assigned to), <span class="cmd">E_VARNF</span> will be raised.

As before, the <span class="cmd">$</span> expression can be used in either <var>start-index-expr</var> or <var>end-index-expr</var>, meaning the length of the original value of the expression just before the <span class="cmd">[...]</span> part.

If <var>start-index-expr</var> or <var>end-index-expr</var> is not an integer, if the value of <var>variable</var> or the property is not a list or string, or <var>result-expr</var> is not the same type as <var>variable</var> or the property, <span class="cmd">E_TYPE</span> is raised. <span class="cmd">E_RANGE</span> is raised if <var>end-index-expr</var> is less than zero or if <var>start-index-expr</var> is greater than the length of the list or string plus one. Note: the length of <var>result-expr</var> does not need to be the same as the length of the specified range.

In precise terms, the subrange assigment

<pre><var>v</var>[<var>start</var>..<var>end</var>] = <var>value</var>
</pre>

is equivalent to

<pre><var>v</var> = {@<var>v</var>[1..<var>start</var> - 1], @<var>value</var>, @<var>v</var>[<var>end</var> + 1..$]}
</pre>

if <var>v</var> is a list and to

<pre><var>v</var> = <var>v</var>[1..<var>start</var> - 1] + <var>value</var> + <var>v</var>[<var>end</var> + 1..$]
</pre>

if <var>v</var> is a string. The assigment expression itself returns the value of <var>result-expr</var>.

<div class="alert alert-info"><span class="label label-info">Note:</span> The use of preceeding a list with the @ symbol is covered in just a bit.</div>

For the following examples, assume that <span class="cmd">l</span> initially contains the list <span class="cmd">{1, 2, 3}</span> and that <span class="cmd">s</span> initially contains the string "foobar":

<pre>l[5..6] = {7, 8}       =>   E_RANGE (error)
l[2..3] = 4            =>   E_TYPE (error)
l[#2..3] = {7}         =>   E_TYPE (error)
s[2..3] = {6}          =>   E_TYPE (error)
l[2..3] = {6, 7, 8, 9} =>   {6, 7, 8, 9}
l                      =>   {1, 6, 7, 8, 9}
l[2..1] = {10, "foo"}  =>   {10, "foo"}
l                      =>   {1, 10, "foo", 6, 7, 8, 9}
l[3][2..$] = "u"       =>   "u"
l                      =>   {1, 10, "fu", 6, 7, 8, 9}
s[7..12] = "baz"       =>   "baz"
s                      =>   "foobarbaz"
s[1..3] = "fu"         =>   "fu"
s                      =>   "fubarbaz"
s[1..0] = "test"       =>   "test"
s                      =>   "testfubarbaz"
</pre>

#### Other Operations on Lists

As was mentioned earlier, lists can be constructed by writing a comma-separated sequence of expressions inside curly braces:

<pre>{<var>expression-1</var>, <var>expression-2</var>, ..., <var>expression-N</var>}
</pre>

The resulting list has the value of <var>expression-1</var> as its first element, that of <var>expression-2</var> as the second, etc.

<pre>{3 < 4, 3 <= 4, 3 >= 4, 3 > 4}  =>  {1, 1, 0, 0}
</pre>

Additionally, one may precede any of these expressions by the splicing operator, <span class="cmd">@</span>. Such an expression must return a list; rather than the old list itself becoming an element of the new list, all of the elements of the old list are included in the new list. This concept is easy to understand, but hard to explain in words, so here are some examples. For these examples, assume that the variable <span class="cmd">a</span> has the value <span class="cmd">{2, 3, 4}</span> and that <span class="cmd">b</span> has the value <span class="cmd">{"Foo", "Bar"}</span>:

<pre>{1, a, 5}   =>  {1, {2, 3, 4}, 5}
{1, @a, 5}  =>  {1, 2, 3, 4, 5}
{a, @a}     =>  {{2, 3, 4}, 2, 3, 4}
{@a, @b}    =>  {2, 3, 4, "Foo", "Bar"}
</pre>

If the splicing operator (<span class="cmd">@</span>) precedes an expression whose value is not a list, then <span class="cmd">E_TYPE</span> is raised as the value of the list construction as a whole.

The list membership expression tests whether or not a given MOO value is an element of a given list and, if so, with what index:

<pre><var>expression-1</var> in <var>expression-2</var>
</pre>

<var>Expression-2</var> must return a list; otherwise, <span class="cmd">E_TYPE</span> is raised. If the value of <var>expression-1</var> is in that list, then the index of its first occurrence in the list is returned; otherwise, the <span class="cmd">in</span> expression returns 0.

<pre>2 in {5, 8, 2, 3}               =>  3
7 in {5, 8, 2, 3}               =>  0
"bar" in {"Foo", "Bar", "Baz"}  =>  2
</pre>

Note that the list membership operator is case-insensitive in comparing strings, just like the comparison operators. To perform a case-sensitive list membership test, use the <span class="cmd">is_member</span> function described later. Note also that since it returns zero only if the given value is not in the given list, the <span class="cmd">in</span> expression can be used either as a membership test or as an element locator.

#### Spreading List Elements Among Variables

It is often the case in MOO programming that you will want to access the elements of a list individually, with each element stored in a separate variables. This desire arises, for example, at the beginning of almost every MOO verb, since the arguments to all verbs are delivered all bunched together in a single list. In such circumstances, you _could_ write statements like these:

<pre>first = args[1];
second = args[2];
if (length(args) > 2)
  third = args[3];
else
  third = 0;
endif
</pre>

This approach gets pretty tedious, both to read and to write, and it's prone to errors if you mistype one of the indices. Also, you often want to check whether or not any _extra_ list elements were present, adding to the tedium.

MOO provides a special kind of assignment expression, called _scattering assignment_ made just for cases such as these. A scattering assignment expression looks like this:

<pre>{<var>target</var>, ...} = <var>expr</var>
</pre>

where each <var>target</var> describes a place to store elements of the list that results from evaluating <var>expr</var>. A <var>target</var> has one of the following forms:

<dl class="dl-horizontal">

<dt><span class="cmd"><var>variable</var></span></dt>

<dd>This is the simplest target, just a simple variable; the list element in the corresponding position is assigned to the variable. This is called a _required_ target, since the assignment is required to put one of the list elements into the variable.</dd>

<dt><span class="cmd">?<var>variable</var></span></dt>

<dd>This is called an _optional_ target, since it doesn't always get assigned an element. If there are any list elements left over after all of the required targets have been accounted for (along with all of the other optionals to the left of this one), then this variable is treated like a required one and the list element in the corresponding position is assigned to the variable. If there aren't enough elements to assign one to this target, then no assignment is made to this variable, leaving it with whatever its previous value was.</dd>

<dt><span class="cmd">?<var>variable</var> = <var>default-expr</var></span></dt>

<dd>This is also an optional target, but if there aren't enough list elements available to assign one to this target, the result of evaluating <var>default-expr</var> is assigned to it instead. Thus, <var>default-expr</var> provides a _default value_ for the variable. The default value expressions are evaluated and assigned working from left to right _after_ all of the other assignments have been performed.</dd>

<dt><span class="cmd">@<var>variable</var></span></dt>

<dd>By analogy with the <span class="cmd">@</span> syntax in list construction, this variable is assigned a list of all of the `leftover' list elements in this part of the list after all of the other targets have been filled in. It is assigned the empty list if there aren't any elements left over. This is called a _rest_ target, since it gets the rest of the elements. There may be at most one rest target in each scattering assignment expression.</dd>

</dl>

If there aren't enough list elements to fill all of the required targets, or if there are more than enough to fill all of the required and optional targets but there isn't a rest target to take the leftover ones, then <span class="cmd">E_ARGS</span> is raised.

Here are some examples of how this works. Assume first that the verb <span class="cmd">me:foo()</span> contains the following code:

<pre>b = c = e = 17;
{a, ?b, ?c = 8, @d, ?e = 9, f} = args;
return {a, b, c, d, e, f};
</pre>

Then the following calls return the given values:

<pre>me:foo(1)                        =>   E_ARGS (error)
me:foo(1, 2)                     =>   {1, 17, 8, {}, 9, 2}
me:foo(1, 2, 3)                  =>   {1, 2, 8, {}, 9, 3}
me:foo(1, 2, 3, 4)               =>   {1, 2, 3, {}, 9, 4}
me:foo(1, 2, 3, 4, 5)            =>   {1, 2, 3, {}, 4, 5}
me:foo(1, 2, 3, 4, 5, 6)         =>   {1, 2, 3, {4}, 5, 6}
me:foo(1, 2, 3, 4, 5, 6, 7)      =>   {1, 2, 3, {4, 5}, 6, 7}
me:foo(1, 2, 3, 4, 5, 6, 7, 8)   =>   {1, 2, 3, {4, 5, 6}, 7, 8}
</pre>

Using scattering assignment, the example at the begining of this section could be rewritten more simply, reliably, and readably:

<pre>{first, second, ?third = 0} = args;
</pre>

<div class="alert alert-info"><span class="label label-info">Fine point:</span> It is good MOO programming style to use a scattering assignment at the top of nearly every verb, since it shows so clearly just what kinds of arguments the verb expects.</div>

#### Getting and Setting the Values of Properties

Usually, one can read the value of a property on an object with a simple expression:

<pre><var>expression</var>.<var>name</var>
</pre>

<var>Expression</var> must return an object number; if not, <span class="cmd">E_TYPE</span> is raised. If the object with that number does not exist, <span class="cmd">E_INVIND</span> is raised. Otherwise, if the object does not have a property with that name, then <span class="cmd">E_PROPNF</span> is raised. Otherwise, if the named property is not readable by the owner of the current verb, then <span class="cmd">E_PERM</span> is raised. Finally, assuming that none of these terrible things happens, the value of the named property on the given object is returned.

I said "usually" in the paragraph above because that simple expression only works if the name of the property obeys the same rules as for the names of variables (i.e., consists entirely of letters, digits, and underscores, and doesn't begin with a digit). Property names are not restricted to this set, though. Also, it is sometimes useful to be able to figure out what property to read by some computation. For these more general uses, the following syntax is also allowed:

<pre><var>expression-1</var>.(<var>expression-2</var>)
</pre>

As before, <var>expression-1</var> must return an object number. <var>Expression-2</var> must return a string, the name of the property to be read; <span class="cmd">E_TYPE</span> is raised otherwise. Using this syntax, any property can be read, regardless of its name.

Note that, as with almost everything in MOO, case is not significant in the names of properties. Thus, the following expressions are all equivalent:

<pre>foo.bar
foo.Bar
foo.("bAr")
</pre>

The LambdaCore database uses several properties on <span class="cmd">#0</span>, the _system object_, for various special purposes. For example, the value of <span class="cmd">#0.room</span> is the "generic room" object, <span class="cmd">#0.exit</span> is the "generic exit" object, etc. This allows MOO programs to refer to these useful objects more easily (and more readably) than using their object numbers directly. To make this usage even easier and more readable, the expression

<pre>$<var>name</var>
</pre>

(where <var>name</var> obeys the rules for variable names) is an abbreviation for

<pre>#0.<var>name</var>
</pre>

Thus, for example, the value <span class="cmd">$nothing</span> mentioned earlier is really <span class="cmd">#-1</span>, the value of <span class="cmd">#0.nothing</span>.

As with variables, one uses the assignment operator (<span class="cmd">=</span>) to change the value of a property. For example, the expression

<pre>14 + (#27.foo = 17)
</pre>

changes the value of the <span class="cmd">foo</span> property of the object numbered 27 to be 17 and then returns 31\. Assignments to properties check that the owner of the current verb has write permission on the given property, raising <span class="cmd">E_PERM</span> otherwise. Read permission is not required.

#### Calling Built-in Functions and Other Verbs

MOO provides a large number of useful functions for performing a wide variety of operations; a complete list, giving their names, arguments, and semantics, appears in a separate section later. As an example to give you the idea, there is a function named <span class="cmd">length</span> that returns the length of a given string or list.

The syntax of a call to a function is as follows:

<pre><var>name</var>(<var>expr-1</var>, <var>expr-2</var>, ..., <var>expr-N</var>)
</pre>

where <var>name</var> is the name of one of the built-in functions. The expressions between the parentheses, called _arguments_, are each evaluated in turn and then given to the named function to use in its appropriate way. Most functions require that a specific number of arguments be given; otherwise, <span class="cmd">E_ARGS</span> is raised. Most also require that certain of the arguments have certain specified types (e.g., the <span class="cmd">length()</span> function requires a list or a string as its argument); <span class="cmd">E_TYPE</span> is raised if any argument has the wrong type.

As with list construction, the splicing operator <span class="cmd">@</span> can precede any argument expression. The value of such an expression must be a list; <span class="cmd">E_TYPE</span> is raised otherwise. The elements of this list are passed as individual arguments, in place of the list as a whole.

Verbs can also call other verbs, usually using this syntax:

<pre><var>expr-0</var>:<var>name</var>(<var>expr-1</var>, <var>expr-2</var>, ..., <var>expr-N</var>)
</pre>

<var>Expr-0</var> must return an object number; <span class="cmd">E_TYPE</span> is raised otherwise. If the object with that number does not exist, <span class="cmd">E_INVIND</span> is raised. If this task is too deeply nested in verbs calling verbs calling verbs, then <span class="cmd">E_MAXREC</span> is raised; the default limit is 50 levels, but this can be changed from within the database; see the chapter on server assumptions about the database for details. If neither the object nor any of its ancestors defines a verb matching the given name, <span class="cmd">E_VERBNF</span> is raised. Otherwise, if none of these nasty things happens, the named verb on the given object is called; the various built-in variables have the following initial values in the called verb:

<dl class="dl-horizontal">

<dt><span class="cmd">this</span></dt>

<dd>an object, the value of <var>expr-0</var></dd>

<dt><span class="cmd">verb</span></dt>

<dd>a string, the <var>name</var> used in calling this verb</dd>

<dt><span class="cmd">args</span></dt>

<dd>a list, the values of <var>expr-1</var>, <var>expr-2</var>, etc.</dd>

<dt><span class="cmd">caller</span></dt>

<dd>an object, the value of <span class="cmd">this</span> in the calling verb</dd>

<dt><span class="cmd">player</span></dt>

<dd>an object, the same value as it had initially in the calling verb or, if the calling verb is running with wizard permissions, the same as the current value in the calling verb.</dd>

</dl>

All other built-in variables (<span class="cmd">argstr</span>, <span class="cmd">dobj</span>, etc.) are initialized with the same values they have in the calling verb.

As with the discussion of property references above, I said "usually" at the beginning of the previous paragraph because that syntax is only allowed when the <var>name</var> follows the rules for allowed variable names. Also as with property reference, there is a syntax allowing you to compute the name of the verb:

<pre><var>expr-0</var>:(<var>expr-00</var>)(<var>expr-1</var>, <var>expr-2</var>, ..., <var>expr-N</var>)
</pre>

The expression <var>expr-00</var> must return a string; <span class="cmd">E_TYPE</span> is raised otherwise.

The splicing operator (<span class="cmd">@</span>) can be used with verb-call arguments, too, just as with the arguments to built-in functions.

In many databases, a number of important verbs are defined on <span class="cmd">#0</span>, the _system object_. As with the <span class="cmd">$foo</span> notation for properties on <span class="cmd">#0</span>, the server defines a special syntax for calling verbs on <span class="cmd">#0</span>:

<pre>$<var>name</var>(<var>expr-1</var>, <var>expr-2</var>, ..., <var>expr-N</var>)
</pre>

(where <var>name</var> obeys the rules for variable names) is an abbreviation for

<pre>#0:<var>name</var>(<var>expr-1</var>, <var>expr-2</var>, ..., <var>expr-N</var>)
</pre>

#### Catching Errors in Expressions

It is often useful to be able to _catch_ an error that an expression raises, to keep the error from aborting the whole task, and to keep on running as if the expression had returned some other value normally. The following expression accomplishes this:

<pre>` <var>expr-1</var> ! <var>codes</var> => <var>expr-2</var> '
</pre>

<div class="well"><span class="label label-info">Note:</span> The open- and close-quotation marks in the previous line are really part of the syntax; you must actually type them as part of your MOO program for this kind of expression.</div>

The <var>codes</var> part is either the keyword <span class="cmd">ANY</span> or else a comma-separated list of expressions, just like an argument list. As in an argument list, the splicing operator (<span class="cmd">@</span>) can be used here. The <span class="cmd">=> <var>expr-2</var></span> part of the error-catching expression is optional.

First, the <var>codes</var> part is evaluated, yielding a list of error codes that should be caught if they're raised; if <var>codes</var> is <span class="cmd">ANY</span>, then it is equivalent to the list of all possible MOO values.

Next, <var>expr-1</var> is evaluated. If it evaluates normally, without raising an error, then its value becomes the value of the entire error-catching expression. If evaluating <var>expr-1</var> results in an error being raised, then call that error <var>E</var>. If <var>E</var> is in the list resulting from evaluating <var>codes</var>, then <var>E</var> is considered _caught_ by this error-catching expression. In such a case, if <var>expr-2</var> was given, it is evaluated to get the outcome of the entire error-catching expression; if <var>expr-2</var> was omitted, then <var>E</var> becomes the value of the entire expression. If <var>E</var> is _not_ in the list resulting from <var>codes</var>, then this expression does not catch the error at all and it continues to be raised, possibly to be caught by some piece of code either surrounding this expression or higher up on the verb-call stack.

Here are some examples of the use of this kind of expression:

<pre>`x + 1 ! E_TYPE => 0'
</pre>

Returns <span class="cmd">x + 1</span> if <span class="cmd">x</span> is an integer, returns <span class="cmd">0</span> if <span class="cmd">x</span> is not an integer, and raises <span class="cmd">E_VARNF</span> if <span class="cmd">x</span> doesn't have a value.

<pre>`x.y ! E_PROPNF, E_PERM => 17'
</pre>

Returns <span class="cmd">x.y</span> if that doesn't cause an error, <span class="cmd">17</span> if <span class="cmd">x</span> doesn't have a <span class="cmd">y</span> property or that property isn't readable, and raises some other kind of error (like <span class="cmd">E_INVIND</span>) if <span class="cmd">x.y</span> does.

<pre>`1 / 0 ! ANY'
</pre>

Returns <span class="cmd">E_DIV</span>.

<div class="alert alert-info"><span class="label label-info">Note:</span> It's important to mention how powerful this compact syntax for writing error catching code can be. When used properly you can write very complex and elegant code. For example imagine that you have a set of objects from different parents, some of which define a specific verb, and some of which do not. If for instance, your code wants to perform some function _if_ the verb exists, you can write `obj:verbname() ! ANY => 0' to allow the MOO to attempt to execute that verb and then if it fails, catch the error and continue operations normally.</div>

#### Parentheses and Operator Precedence

As shown in a few examples above, MOO allows you to use parentheses to make it clear how you intend for complex expressions to be grouped. For example, the expression

<pre>3 * (4 + 5)
</pre>

performs the addition of 4 and 5 before multiplying the result by 3.

If you leave out the parentheses, MOO will figure out how to group the expression according to certain rules. The first of these is that some operators have higher _precedence_ than others; operators with higher precedence will more tightly bind to their operands than those with lower precedence. For example, multiplication has higher precedence than addition; thus, if the parentheses had been left out of the expression in the previous paragraph, MOO would have grouped it as follows:

<pre>(3 * 4) + 5
</pre>

The table below gives the relative precedence of all of the MOO operators; operators on higher lines in the table have higher precedence and those on the same line have identical precedence:

<pre>!       - (without a left operand)
^
*       /       %
+       -
==      !=      <       <=      >       >=      in
&&      ||
... ? ... | ... (the conditional expression)
=
</pre>

Thus, the horrendous expression

<pre>x = a < b && c > d + e * f ? w in y | - q - r
</pre>

would be grouped as follows:

<pre>x = (((a < b) && (c > (d + (e * f)))) ? (w in y) | ((- q) - r))
</pre>

It is best to keep expressions simpler than this and to use parentheses liberally to make your meaning clear to other humans.

### MOO Language Statements

Statements are MOO constructs that, in contrast to expressions, perform some useful, non-value-producing operation. For example, there are several kinds of statements, called _looping constructs_, that repeatedly perform some set of operations. Fortunately, there are many fewer kinds of statements in MOO than there are kinds of expressions.

#### Errors While Executing Statements

Statements do not return values, but some kinds of statements can, under certain circumstances described below, generate errors. If such an error is generated in a verb whose <span class="cmd">d</span> (debug) bit is not set, then the error is ignored and the statement that generated it is simply skipped; execution proceeds with the next statement.

<div class="well"><span class="label label-info">Note:</span> This error-ignoring behavior is very error prone, since it affects _all_ errors, including ones the programmer may not have anticipated. The <span class="cmd">d</span> bit exists only for historical reasons; it was once the only way for MOO programmers to catch and handle errors. The error-catching expression and the <span class="cmd">try</span>-<span class="cmd">except</span> statement are far better ways of accomplishing the same thing.</div>

If the <span class="cmd">d</span> bit is set, as it usually is, then the error is _raised_ and can be caught and handled either by code surrounding the expression in question or by verbs higher up on the chain of calls leading to the current verb. If the error is not caught, then the server aborts the entire task and, by default, prints a message to the current player. See the descriptions of the error-catching expression and the <span class="cmd">try</span>-<span class="cmd">except</span> statement for the details of how errors can be caught, and the chapter on server assumptions about the database for details on the handling of uncaught errors.

#### Simple Statements

The simplest kind of statement is the _null_ statement, consisting of just a semicolon:

<pre>;
</pre>

It doesn't do anything at all, but it does it very quickly.

The next simplest statement is also one of the most common, the expression statement, consisting of any expression followed by a semicolon:

<pre><var>expression</var>;
</pre>

The given expression is evaluated and the resulting value is ignored. Commonly-used kinds of expressions for such statements include assignments and verb calls. Of course, there's no use for such a statement unless the evaluation of <var>expression</var> has some side-effect, such as changing the value of some variable or property, printing some text on someone's screen, etc.

<pre>#42.weight = 40;
#42.weight;
2 + 5;
obj:verbname();
1 > 2;
2 < 1;
</pre>

#### Statements for Testing Conditions

The <span class="cmd">if</span> statement allows you to decide whether or not to perform some statements based on the value of an arbitrary expression:

<pre>if (<var>expression</var>)
  <var>statements</var>
endif
</pre>

<var>Expression</var> is evaluated and, if it returns a true value, the statements are executed in order; otherwise, nothing more is done.

One frequently wants to perform one set of statements if some condition is true and some other set of statements otherwise. The optional <span class="cmd">else</span> phrase in an <span class="cmd">if</span> statement allows you to do this:

<pre>if (<var>expression</var>)
  <var>statements-1</var>
else
  <var>statements-2</var>
endif
</pre>

This statement is executed just like the previous one, except that <var>statements-1</var> are executed if <var>expression</var> returns a true value and <var>statements-2</var> are executed otherwise.

Sometimes, one needs to test several conditions in a kind of nested fashion:

<pre>if (<var>expression-1</var>)
  <var>statements-1</var>
else
  if (<var>expression-2</var>)
    <var>statements-2</var>
  else
    if (<var>expression-3</var>)
      <var>statements-3</var>
    else
      <var>statements-4</var>
    endif
  endif
endif
</pre>

Such code can easily become tedious to write and difficult to read. MOO provides a somewhat simpler notation for such cases:

<pre>if (<var>expression-1</var>)
  <var>statements-1</var>
elseif (<var>expression-2</var>)
  <var>statements-2</var>
elseif (<var>expression-3</var>)
  <var>statements-3</var>
else
  <var>statements-4</var>
endif
</pre>

Note that <span class="cmd">elseif</span> is written as a single word, without any spaces. This simpler version has the very same meaning as the original: evaluate <var>expression-i</var> for <var>i</var> equal to 1, 2, and 3, in turn, until one of them returns a true value; then execute the <var>statements-i</var> associated with that expression. If none of the <var>expression-i</var> return a true value, then execute <var>statements-4</var>.

Any number of <span class="cmd">elseif</span> phrases can appear, each having this form:

<pre>elseif (<var>expression</var>) 
    <var>statements</var>
</pre>

The complete syntax of the <span class="cmd">if</span> statement, therefore, is as follows:

<pre>if (<var>expression</var>)
  <var>statements</var>
<var>zero-or-more-elseif-phrases</var>
<var>an-optional-else-phrase</var>
endif
</pre>

#### Statements for Looping

MOO provides three different kinds of looping statements, allowing you to have a set of statements executed (1) once for each element of a given list, (2) once for each integer or object number in a given range, and (3) over and over until a given condition stops being true.

##### The for-in loop

<div class="alert alert-info"><span class="label label-info">Note:</span> In some programming languages this is referred to as a foreach loop. The syntax and usage is roughly the same.</div>

To perform some statements once for each element of a given list, use this syntax:

<pre>for <var>variable</var> in (<var>expression</var>)
  <var>statements</var>
endfor
</pre>

The expression is evaluated and should return a list; if it does not, <span class="cmd">E_TYPE</span> is raised. The <var>statements</var> are then executed once for each element of that list in turn; each time, the given <var>variable</var> is assigned the value of the element in question. For example, consider the following statements:

<pre>odds = {1, 3, 5, 7, 9};
evens = {};
for n in (odds)
  evens = {@evens, n + 1};
endfor
</pre>

The value of the variable <span class="cmd">evens</span> after executing these statements is the list

<pre>{2, 4, 6, 8, 10}
</pre>

Another example of this, looping over all the children of an object:

<pre>for child in (children(obj))
    notify(player, tostr(o.name, " is located in ", o.location));
endfor
</pre>

Another exmaple of this, looping over a list of strings:

<pre>strings = {"foo", "bar", "baz"};
for string in (strings)
    notify(player, string);
endfor
</pre>

##### The For-Range Loop

To perform a set of statements once for each integer or object number in a given range, use this syntax:

<pre>for <var>variable</var> in [<var>expression-1</var>..<var>expression-2</var>]
  <var>statements</var>
endfor
</pre>

The two expressions are evaluated in turn and should either both return integers or both return object numbers; <span class="cmd">E_TYPE</span> is raised otherwise. The <var>statements</var> are then executed once for each integer (or object number, as appropriate) greater than or equal to the value of <var>expression-1</var> and less than or equal to the result of <var>expression-2</var>, in increasing order. Each time, the given variable is assigned the integer or object number in question. For example, consider the following statements:

<pre>evens = {};
for n in [1..5]
  evens = {@evens, 2 * n};
endfor
</pre>

The value of the variable <span class="cmd">evens</span> after executing these statements is just as in the previous example: the list

<pre>{2, 4, 6, 8, 10}
</pre>

The following loop over object numbers prints out the number and name of every valid object in the database:

<pre>for o in [#0..max_object()]
  if (valid(o))
    notify(player, tostr(o, ": ", o.name));
  endif
endfor
</pre>

##### The While Loop

The final kind of loop in MOO executes a set of statements repeatedly as long as a given condition remains true:

<pre>while (<var>expression</var>)
  <var>statements</var>
endwhile
</pre>

The expression is evaluated and, if it returns a true value, the <var>statements</var> are executed; then, execution of the <span class="cmd">while</span> statement begins all over again with the evaluation of the expression. That is, execution alternates between evaluating the expression and executing the statements until the expression returns a false value. The following example code has precisely the same effect as the loop just shown above:

<pre>evens = {};
n = 1;
while (n <= 5)
  evens = {@evens, 2 * n};
  n = n + 1;
endwhile
</pre>

<div class="alert alert-info"><span class="label label-info">Fine point:</span> It is also possible to give a _name_ to a <span class="cmd">while</span> loop.</div>

<pre>while <var>name</var> (<var>expression</var>)
  <var>statements</var>
endwhile
</pre>

which has precisely the same effect as

<pre>while (<var>name</var> = <var>expression</var>)
  <var>statements</var>
endwhile
</pre>

This naming facility is only really useful in conjunction with the <span class="cmd">break</span> and <span class="cmd">continue</span> statements, described in the next section.

With each kind of loop, it is possible that the statements in the body of the loop will never be executed at all. For iteration over lists, this happens when the list returned by the expression is empty. For iteration on integers, it happens when <var>expression-1</var> returns a larger integer than <var>expression-2</var>. Finally, for the <span class="cmd">while</span> loop, it happens if the expression returns a false value the very first time it is evaluated.

<div class="alert alert-warning"><span class="label label-warning">Warning:</span> With <span class="cmd">while</span> loops it is especially important to make sure you do not create an infinite loop. That is, a loop that will never terminate because it's expression will never become false.</div>

#### Terminating One or All Iterations of a Loop

Sometimes, it is useful to exit a loop before it finishes all of its iterations. For example, if the loop is used to search for a particular kind of element of a list, then it might make sense to stop looping as soon as the right kind of element is found, even if there are more elements yet to see. The <span class="cmd">break</span> statement is used for this purpose; it has the form

<pre>break;
</pre>

or

<pre>break <var>name</var>;
</pre>

Each <span class="cmd">break</span> statement indicates a specific surrounding loop; if <var>name</var> is not given, the statement refers to the innermost one. If it is given, <var>name</var> must be the name appearing right after the <span class="cmd">for</span> or <span class="cmd">while</span> keyword of the desired enclosing loop. When the <span class="cmd">break</span> statement is executed, the indicated loop is immediately terminated and executing continues just as if the loop had completed its iterations normally.

MOO also allows you to terminate just the current iteration of a loop, making it immediately go on to the next one, if any. The <span class="cmd">continue</span> statement does this; it has precisely the same forms as the <span class="cmd">break</span> statement:

<pre>continue;
</pre>

or

<pre>continue <var>name</var>;
</pre>

An example that sums up a list of integers, excluding any integer equal to four:

<pre>my_list = {1, 2, 3, 4, 5, 6, 7};
sum = 0;
for element in (my_list)
    if (element == 4)
        continue;
    endif
    sum = sum + element;
endfor
</pre>

An example that

<pre>my_list = {#13633, #98, #15840, #18657, #22664};
i = 0;
found = 0;
for obj in (my_list)
    i = i + 1;
    if (obj == #18657)
        found = 1;
        break;
    endif
endfor
if (found)
    notify(player, tostr("found #18657 at ", i, " index"));
else
    notify(player, "couldn't find #18657 in the list!");
endif
</pre>

#### Returning a Value from a Verb

The MOO program in a verb is just a sequence of statements. Normally, when the verb is called, those statements are simply executed in order and then the integer 0 is returned as the value of the verb-call expression. Using the <span class="cmd">return</span> statement, one can change this behavior. The <span class="cmd">return</span> statement has one of the following two forms:

<pre>return;
</pre>

or

<pre>return <var>expression</var>;
</pre>

When it is executed, execution of the current verb is terminated immediately after evaluating the given <var>expression</var>, if any. The verb-call expression that started the execution of this verb then returns either the value of <var>expression</var> or the integer 0, if no <var>expression</var> was provided.

We could modify the example given above. Imagine a verb called <var>has_object</var> which takes an object (that we want to find) as it's first argument and a list of objects (to search) as it's second argument:

<pre>{seek_obj, list_of_objects} = args;
for obj in (list_of_objects)
    if (obj == seek_obj)
        return 1;
    endif
endfor
</pre>

The verb above could be called with <span class="cmd"><var>obj_with_verb</var>:<var>has_object</var>(<var>#18657</var>, <var>{#1, #3, #4, #3000}</var>)</span> and it would return <span class="cmd">false</span> (0) if the object was not found in the list. It would return <span class="cmd">true</span> (1) if the object was found in the list.

Of course we could write this much simplier (and get the index of the object in the list at the same time):

<pre>{seek_obj, list_of_objects} = args;
return seek_obj in list_of_objects;
</pre>

#### Handling Errors in Statements

Normally, whenever a piece of MOO code raises an error, the entire task is aborted and a message printed to the user. Often, such errors can be anticipated in advance by the programmer and code written to deal with them in a more graceful manner. The <span class="cmd">try</span>-<span class="cmd">except</span> statement allows you to do this; the syntax is as follows:

<pre>try
  <var>statements-0</var>
except <var>variable-1</var> (<var>codes-1</var>)
  <var>statements-1</var>
except <var>variable-2</var> (<var>codes-2</var>)
  <var>statements-2</var>
...
endtry
</pre>

where the <var>variable</var>s may be omitted and each <var>codes</var> part is either the keyword <span class="cmd">ANY</span> or else a comma-separated list of expressions, just like an argument list. As in an argument list, the splicing operator (<span class="cmd">@</span>) can be used here. There can be anywhere from 1 to 255 <span class="cmd">except</span> clauses.

First, each <var>codes</var> part is evaluated, yielding a list of error codes that should be caught if they're raised; if a <var>codes</var> is <span class="cmd">ANY</span>, then it is equivalent to the list of all possible MOO values.

Next, <var>statements-0</var> is executed; if it doesn't raise an error, then that's all that happens for the entire <span class="cmd">try</span>-<span class="cmd">except</span> statement. Otherwise, let <var>E</var> be the error it raises. From top to bottom, <var>E</var> is searched for in the lists resulting from the various <var>codes</var> parts; if it isn't found in any of them, then it continues to be raised, possibly to be caught by some piece of code either surrounding this <span class="cmd">try</span>-<span class="cmd">except</span> statement or higher up on the verb-call stack.

If <var>E</var> is found first in <var>codes-i</var>, then <var>variable-i</var> (if provided) is assigned a value containing information about the error being raised and <var>statements-i</var> is executed. The value assigned to <var>variable-i</var> is a list of four elements:

<pre>{<var>code</var>, <var>message</var>, <var>value</var>, <var>traceback</var>}
</pre>

where <var>code</var> is <var>E</var>, the error being raised, <var>message</var> and <var>value</var> are as provided by the code that raised the error, and <var>traceback</var> is a list like that returned by the <span class="cmd">callers()</span> function, including line numbers. The <var>traceback</var> list contains entries for every verb from the one that raised the error through the one containing this <span class="cmd">try</span>-<span class="cmd">except</span> statement.

Unless otherwise mentioned, all of the built-in errors raised by expressions, statements, and functions provide <span class="cmd">tostr(<var>code</var>)</span> as <var>message</var> and zero as <var>value</var>.

Here's an example of the use of this kind of statement:

<pre>try
  result = object:(command)(@arguments);
  player:tell("=> ", toliteral(result));
except v (ANY)
  tb = v[4];
  if (length(tb) == 1)
    player:tell("** Illegal command: ", v[2]);
  else
    top = tb[1];
    tb[1..1] = {};
    player:tell(top[1], ":", top[2], ", line ", top[6], ":", v[2]);
    for fr in (tb)
      player:tell("... called from ", fr[1], ":", fr[2], ", line ", fr[6]);
    endfor
    player:tell("(End of traceback)");
  endif
endtry
</pre>

#### Cleaning Up After Errors

Whenever an error is raised, it is usually the case that at least some MOO code gets skipped over and never executed. Sometimes, it's important that a piece of code _always_ be executed, whether or not an error is raised. Use the <span class="cmd">try</span>-<span class="cmd">finally</span> statement for these cases; it has the following syntax:

<pre>try
  <var>statements-1</var>
finally
  <var>statements-2</var>
endtry
</pre>

First, <var>statements-1</var> is executed; if it completes without raising an error, returning from this verb, or terminating the current iteration of a surrounding loop (we call these possibilities _transferring control_), then <var>statements-2</var> is executed and that's all that happens for the entire <span class="cmd">try</span>-<span class="cmd">finally</span> statement.

Otherwise, the process of transferring control is interrupted and <var>statments-2</var> is executed. If <var>statements-2</var> itself completes without transferring control, then the interrupted control transfer is resumed just where it left off. If <var>statements-2</var> does transfer control, then the interrupted transfer is simply forgotten in favor of the new one.

In short, this statement ensures that <var>statements-2</var> is executed after control leaves <var>statements-1</var> for whatever reason; it can thus be used to make sure that some piece of cleanup code is run even if <var>statements-1</var> doesn't simply run normally to completion.

Here's an example:

<pre>try
  start = time();
  object:(command)(@arguments);
finally
  end = time();
  this:charge_user_for_seconds(player, end - start);
endtry
</pre>

#### Executing Statements at a Later Time

It is sometimes useful to have some sequence of statements execute at a later time, without human intervention. For example, one might implement an object that, when thrown into the air, eventually falls back to the ground; the <span class="cmd">throw</span> verb on that object should arrange to print a message about the object landing on the ground, but the message shouldn't be printed until some number of seconds have passed.

The <span class="cmd">fork</span> statement is intended for just such situations and has the following syntax:

<pre>fork (<var>expression</var>)
  <var>statements</var>
endfork
</pre>

The <span class="cmd">fork</span> statement first executes the expression, which must return a integer; call that integer <var>n</var>. It then creates a new MOO _task_ that will, after at least <var>n</var> seconds, execute the statements. When the new task begins, all variables will have the values they had at the time the <span class="cmd">fork</span> statement was executed. The task executing the <span class="cmd">fork</span> statement immediately continues execution. The concept of tasks is discussed in detail in the next section.

By default, there is no limit to the number of tasks any player may fork, but such a limit can be imposed from within the database. See the chapter on server assumptions about the database for details.

Occasionally, one would like to be able to kill a forked task before it even starts; for example, some player might have caught the object that was thrown into the air, so no message should be printed about it hitting the ground. If a variable name is given after the <span class="cmd">fork</span> keyword, like this:

<pre>fork <var>name</var> (<var>expression</var>)
  <var>statements</var>
endfork
</pre>

then that variable is assigned the _task ID_ of the newly-created task. The value of this variable is visible both to the task executing the fork statement and to the statements in the newly-created task. This ID can be passed to the <span class="cmd">kill_task()</span> function to keep the task from running and will be the value of <span class="cmd">task_id()</span> once the task begins execution.

<div class="alert alert-info"><span class="label label-info">Note:</span> This feature has other uses as well. The MOO is single threaded, which means that complex logic (verbs that call verbs that call verbs ...) can cause the MOO to _lag_. For instance, let's say when your user tosses their ball up, you want to calculate a complex trejectory involve the ball and other objects in the room. These calculations are costly and done in another verb, they take time to be performed. However, you want some actions to happen both before the calculations (everyone in the room seeing the ball is thrown into the air) and after the ball has left the players hands (the player reaches into their pocket and pulls out a new ball). If there is no <span class="cmd">fork()</span> then the calculations need to complete before the verb can continue execution, which means the player won't pull out a fresh ball until after the calculations are complete. A <span class="cmd">fork()</span> allows the player to throw the ball, the MOO to <span class="cmd">fork()</span> the task, which allows execution of the verb to continue right away and the user to pull out a new ball, without experiencing the delay that the calculations being returned (without a <span class="cmd">fork()</span>) would have inccured.</div>

An example of this:

<pre>{ball} = args;
player:tell("You throw the ball!");
ball:calculate_trajectory();
player:tell("You get out another ball!");
</pre>

In the above example, <span class="cmd">player:tell("You get out another ball!");</span> will not be executed until after <span class="cmd">ball:calculate_trajectory();</span> is completed.

<pre>{ball} = args;
player:tell("You throw the ball!");
fork (1)
    ball:calculate_trajectory();
endfor
player:tell("You get out another ball!");
</pre>

In this forked example, the ball will be thrown, the task forked for 1 second later and the the final line telling the player they got out another ball will be followed up right after, without having to wait for the trajectory verb to finish running.

This type of fork cannot be used if the trajectory is required by the code that runs after it. For instance:

<pre>{ball} = args;
player:tell("You throw the ball!");
direction = ball:calculate_trajectory();
player:tell("You get out another ball!");
player:tell("Your ball arcs to the " + direction);
</pre>

If the above task was forked as it is below:

<pre>{ball} = args;
player:tell("You throw the ball!");
fork (1)
    direction = ball:calculate_trajectory();
endfork
player:tell("You get out another ball!");
player:tell("Your ball arcs to the " + direction);
</pre>

The verb would raise <span class="cmd">E_VARNF</span> due to direction not being defined.

### MOO Tasks

A _task_ is an execution of a MOO program. There are five kinds of tasks in LambdaMOO:

*   Every time a player types a command, a task is created to execute that command; we call these _command tasks_.
*   Whenever a player connects or disconnects from the MOO, the server starts a task to do whatever processing is necessary, such as printing out <span class="cmd">Munchkin has connected</span> to all of the players in the same room; these are called _server tasks_.
*   The <span class="cmd">fork</span> statement in the programming language creates a task whose execution is delayed for at least some given number of seconds; these are _forked tasks_.
*   The <span class="cmd">suspend()</span> function suspends the execution of the current task. A snapshot is taken of whole state of the execution, and the execution will be resumed later. These are called _suspended tasks_.
*   The <span class="cmd">read()</span> function also suspends the execution of the current task, in this case waiting for the player to type a line of input. When the line is received, the task resumes with the <span class="cmd">read()</span> function returning the input line as result. These are called _reading tasks_.

The last three kinds of tasks above are collectively known as _queued tasks_ or _background tasks_, since they may not run immediately.

To prevent a maliciously- or incorrectly-written MOO program from running forever and monopolizing the server, limits are placed on the running time of every task. One limit is that no task is allowed to run longer than a certain number of seconds; command and server tasks get five seconds each while other tasks get only three seconds. This limit is, in practice, rarely reached. The reason is that there is also a limit on the number of operations a task may execute.

The server counts down _ticks_ as any task executes. Roughly speaking, it counts one tick for every expression evaluation (other than variables and literals), one for every <span class="cmd">if</span>, <span class="cmd">fork</span> or <span class="cmd">return</span> statement, and one for every iteration of a loop. If the count gets all the way down to zero, the task is immediately and unceremoniously aborted. By default, command and server tasks begin with an store of 30,000 ticks; this is enough for almost all normal uses. Forked, suspended, and reading tasks are allotted 15,000 ticks each.

These limits on seconds and ticks may be changed from within the database, as can the behavior of the server after it aborts a task for running out; see the chapter on server assumptions about the database for details.

Because queued tasks may exist for long periods of time before they begin execution, there are functions to list the ones that you own and to kill them before they execute. These functions, among others, are discussed in the following section.

### Built-in Functions

There are a large number of built-in functions available for use by MOO programmers. Each one is discussed in detail in this section. The presentation is broken up into subsections by grouping together functions with similar or related uses.

For most functions, the expected types of the arguments are given; if the actual arguments are not of these types, <span class="cmd">E_TYPE</span> is raised. Some arguments can be of any type at all; in such cases, no type specification is given for the argument. Also, for most functions, the type of the result of the function is given. Some functions do not return a useful result; in such cases, the specification <span class="cmd">none</span> is used. A few functions can potentially return any type of value at all; in such cases, the specification <span class="cmd">value</span> is used.

Most functions take a certain fixed number of required arguments and, in some cases, one or two optional arguments. If a function is called with too many or too few arguments, <span class="cmd">E_ARGS</span> is raised.

Functions are always called by the program for some verb; that program is running with the permissions of some player, usually the owner of the verb in question (it is not always the owner, though; wizards can use <span class="cmd">set_task_perms()</span> to change the permissions _on the fly_). In the function descriptions below, we refer to the player whose permissions are being used as the _programmer_.

Many built-in functions are described below as raising <span class="cmd">E_PERM</span> unless the programmer meets certain specified criteria. It is possible to restrict use of any function, however, so that only wizards can use it; see the chapter on server assumptions about the database for details.

#### Object-Oriented Programming

One of the most important facilities in an object-oriented programming language is ability for a child object to make use of a parent's implementation of some operation, even when the child provides its own definition for that operation. The <span class="cmd">pass()</span> function provides this facility in MOO.

<article class="well bg-info text-info" id="function-pass">**<u>function:</u> <span class="cmd">pass</span>**

pass -- calls the verb with the same name as the current verb but as defined on the parent of the object that defines the current verb.

<div class="well bg-info"><var>value</var> <span class="cmd">pass</span> (<var>arg</var>, ...)</div>

Often, it is useful for a child object to define a verb that _augments_ the behavior of a verb on its parent object. For example, in the LambdaCore database, the root object (which is an ancestor of every other object) defines a verb called <span class="cmd">description</span> that simply returns the value of <span class="cmd">this.description</span>; this verb is used by the implementation of the <span class="cmd">look</span> command. In many cases, a programmer would like the description of some object to include some non-constant part; for example, a sentence about whether or not the object was `awake' or `sleeping'. This sentence should be added onto the end of the normal description. The programmer would like to have a means of calling the normal <span class="cmd">description</span> verb and then appending the sentence onto the end of that description. The function <span class="cmd">pass()</span> is for exactly such situations.

<span class="cmd">pass</span> calls the verb with the same name as the current verb but as defined on the parent of the object that defines the current verb. The arguments given to <span class="cmd">pass</span> are the ones given to the called verb and the returned value of the called verb is returned from the call to <span class="cmd">pass</span>. The initial value of <span class="cmd">this</span> in the called verb is the same as in the calling verb.

Thus, in the example above, the child-object's <span class="cmd">description</span> verb might have the following implementation:

<pre>return pass() + "  It is " + (this.awake ? "awake." | "sleeping.");
</pre>

That is, it calls its parent's <span class="cmd">description</span> verb and then appends to the result a sentence whose content is computed based on the value of a property on the object.

In almost all cases, you will want to call <span class="cmd">pass()</span> with the same arguments as were given to the current verb. This is easy to write in MOO; just call <span class="cmd">pass(@args)</span>.

</article>

#### Manipulating MOO Values

There are several functions for performing primitive operations on MOO values, and they can be cleanly split into two kinds: those that do various very general operations that apply to all types of values, and those that are specific to one particular type. There are so many operations concerned with objects that we do not list them in this section but rather give them their own section following this one.

##### General Operations Applicable to all Values

<article class="well bg-info text-info" id="function-typeof">**<u>Function:</u> <span class="cmd">typeof</span>**

typeof -- Takes any MOO value and returns an integer representing the type of <var>value</var>.

<div class="well bg-info"><var>int</var> <span class="cmd">typeof</span> (<var>value</var>)</div>

The result is the same as the initial value of one of these built-in variables: <span class="cmd">INT</span>, <span class="cmd">FLOAT</span>, <span class="cmd">STR</span>, <span class="cmd">LIST</span>, <span class="cmd">OBJ</span>, or <span class="cmd">ERR</span>. Thus, one usually writes code like this:

<pre>if (typeof(x) == LIST) ...
</pre>

and not like this:

<pre>if (typeof(x) == 3) ...
</pre>

because the former is much more readable than the latter.

</article>

<article class="well bg-info text-info" id="function-tostr">**<u>Function:</u> <span class="cmd">tostr</span>**

tostr -- Converts all of the given MOO values into strings and returns the concatenation of the results.

<div class="well bg-info"><var>str</var> <span class="cmd">tostr</span> (<var>value</var>, ...)</div>

<pre>tostr(17)                  =>   "17"
tostr(1.0/3.0)             =>   "0.333333333333333"
tostr(#17)                 =>   "#17"
tostr("foo")               =>   "foo"
tostr({1, 2})              =>   "{list}"
tostr(E_PERM)              =>   "Permission denied"
tostr("3 + 4 = ", 3 + 4)   =>   "3 + 4 = 7"
</pre>

<div class="alert alert-warning"><span class="label label-warning">Warning</span> <span class="cmd">tostr()</span> does not do a good job of converting lists into strings; all lists, including the empty list, are converted into the string <span class="cmd">"{list}"</span>. The function <span class="cmd">toliteral()</span>, below, is better for this purpose.</div>

</article>

<article class="well bg-info text-info" id="function-toliteral">**<u>Function:</u> <span class="cmd">toliteral</span>**

Returns a string containing a MOO literal expression that, when evaluated, would be equal to <var>value</var>.

<div class="well bg-info"><var>str</var> <span class="cmd">toliteral</span> (<var>value</var>)</div>

<pre>toliteral(17)         =>   "17"
toliteral(1.0/3.0)    =>   "0.333333333333333"
toliteral(#17)        =>   "#17"
toliteral("foo")      =>   "\"foo\""
toliteral({1, 2})     =>   "{1, 2}"
toliteral(E_PERM)     =>   "E_PERM"
</pre>

</article>

<article class="well bg-info text-info" id="function-toint">**<u>Function:</u> <span class="cmd">toint</span>**  
**<u>Function:</u> <span class="cmd">tonum</span>**

toint -- Converts the given MOO value into an integer and returns that integer.

<div class="well bg-info"><var>int</var> <span class="cmd">toint</span> (<var>value</var>)</div>

Floating-point numbers are rounded toward zero, truncating their fractional parts. Object numbers are converted into the equivalent integers. Strings are parsed as the decimal encoding of a real number which is then converted to an integer. Errors are converted into integers obeying the same ordering (with respect to <span class="cmd"><=</span> as the errors themselves. <span class="cmd">toint()</span> raises <span class="cmd">E_TYPE</span> if <var>value</var> is a list. If <var>value</var> is a string but the string does not contain a syntactically-correct number, then <span class="cmd">toint()</span> returns 0.

<pre>toint(34.7)        =>   34
toint(-34.7)       =>   -34
toint(#34)         =>   34
toint("34")        =>   34
toint("34.7")      =>   34
toint(" - 34  ")   =>   -34
toint(E_TYPE)      =>   1
</pre>

</article>

<article class="well bg-info text-info" id="function-toobj">**<u>Function:</u> <span class="cmd">toobj</span>**

toobj -- Converts the given MOO value into an object number and returns that object number.

<div class="well bg-info"><var>obj</var> <span class="cmd">toobj</span> (<var>value</var>)</div>

The conversions are very similar to those for <span class="cmd">toint()</span> except that for strings, the number _may_ be preceded by <span class="cmd">#</span>.

<pre>toobj("34")       =>   #34
toobj("#34")      =>   #34
toobj("foo")      =>   #0
toobj({1, 2})     => E_TYPE (error)
</pre>

</article>

<article class="well bg-info text-info" id="function-tofloat">**<u>Function:</u> <span class="cmd">tofloat</span>**

tofloat -- Converts the given MOO value into a floating-point number and returns that number.

<div class="well bg-info"><var>float</var> <span class="cmd">tofloat</span> (<var>value</var>)</div>

Integers and object numbers are converted into the corresponding integral floating-point numbers. Strings are parsed as the decimal encoding of a real number which is then represented as closely as possible as a floating-point number. Errors are first converted to integers as in <span class="cmd">toint()</span> and then converted as integers are. <span class="cmd">tofloat()</span> raises <span class="cmd">E_TYPE</span> if <var>value</var> is a list. If <var>value</var> is a string but the string does not contain a syntactically-correct number, then <span class="cmd">tofloat()</span> returns 0.

<pre>tofloat(34)          =>   34.0
tofloat(#34)         =>   34.0
tofloat("34")        =>   34.0
tofloat("34.7")      =>   34.7
tofloat(E_TYPE)      =>   1.0
</pre>

</article>

<article class="well bg-info text-info" id="function-equal">**<u>Function:</u> <span class="cmd">equal</span>**

equal -- Returns true if <var>value1</var> is completely indistinguishable from <var>value2</var>.

<div class="well bg-info"><var>int</var> <span class="cmd">equal</span> (<var>value</var>, <var>value2</var>)</div>

This is much the same operation as <span class="cmd"><var>value1</var> == <var>value2</var></span> except that, unlike <span class="cmd">==</span>, the <span class="cmd">equal()</span> function does not treat upper- and lower-case characters in strings as equal and thus, is case-sensitive.

<pre>"Foo" == "foo"         =>   1
equal("Foo", "foo")    =>   0
equal("Foo", "Foo")    =>   1
</pre>

</article>

<article class="well bg-info text-info" id="function-value-bytes">**<u>Function:</u> <span class="cmd">value_bytes</span>**

value_bytes -- Returns the number of bytes of the server's memory required to store the given <var>value</var>.

<div class="well bg-info"><var>int</var> <span class="cmd">value_bytes</span> (<var>value</var>)</div>

</article>

<article class="well bg-info text-info" id="function-string-hash">**<u>Function:</u> <span class="cmd">value_hash</span>**

value_hash -- Returns the same string as <span class="cmd">string_hash(toliteral(<var>value</var>))</span>.

<div class="well bg-info"><var>str</var> <span class="cmd">value_hash</span> (<var>value</var>)</div>

See the description of <span class="cmd">string_hash()</span> for details.

</article>

##### Operations on Numbers

<article class="well bg-info text-info" id="function-random">**<u>Function:</u> <span class="cmd">random</span>**

random -- An integer is chosen randomly from the range <span class="cmd">[1..<var>mod</var>]</span> and returned.

<div class="well bg-info"><var>int</var> <span class="cmd">random</span> ([int <var>mod</var>])</div>

<var>mod</var> must be a positive integer; otherwise, <span class="cmd">E_INVARG</span> is raised. If <var>mod</var> is not provided, it defaults to the largest MOO integer, 2147483647.

<div class="alert alert-warning"><span class="label label-warning">Warning:</span> The <span class="cmd">random()</span> function is not very random. You should augment it's randomness with something like this: <span class="cmd">random() % 100 + 1</span> for better randomness.</div>

</article>

<article class="well bg-info text-info" id="function-min">**<u>Function:</u> <span class="cmd">min</span>**

min -- Return the smallest of it's arguments.

<div class="well bg-info"><var>int</var> <span class="cmd">min</span> (int <var>x</var>, ...)</div>

All of the arguments must be numbers of the same kind (i.e., either integer or floating-point); otherwise <span class="cmd">E_TYPE</span> is raised.

</article>

<article class="well bg-info text-info" id="function-max">**<u>Function:</u> <span class="cmd">max</span>**

max -- Return the largest of it's arguments.

<div class="well bg-info"><var>int</var> <span class="cmd">max</span> (int <var>x</var>, ...)</div>

All of the arguments must be numbers of the same kind (i.e., either integer or floating-point); otherwise <span class="cmd">E_TYPE</span> is raised.

</article>

<article class="well bg-info text-info" id="function-abs">**<u>Function:</u> <span class="cmd">abs</span>**

abs -- Returns the absolute value of <var>x</var>.

<div class="well bg-info"><var>int</var> <span class="cmd">abs</span> (int <var>x</var>)</div>

If <var>x</var> is negative, then the result is <span class="cmd">-<var>x</var></span>; otherwise, the result is <var>x</var>. The number <var>x</var> can be either integer or floating-point; the result is of the same kind.

</article>

<article class="well bg-info text-info" id="function-floatstr">**<u>Function:</u> <span class="cmd">floatstr</span>**

floatstr -- Converts <var>x</var> into a string with more control than provided by either <span class="cmd">tostr()</span> or <span class="cmd">toliteral()</span>.

<div class="well bg-info"><var>str</var> <span class="cmd">floatstr</span> (float <var>x</var>, int <var>precision</var> [, <var>scientific</var>])</div>

<var>Precision</var> is the number of digits to appear to the right of the decimal point, capped at 4 more than the maximum available precision, a total of 19 on most machines; this makes it possible to avoid rounding errors if the resulting string is subsequently read back as a floating-point value. If <var>scientific</var> is false or not provided, the result is a string in the form <span class="cmd">"MMMMMMM.DDDDDD"</span>, preceded by a minus sign if and only if <var>x</var> is negative. If <var>scientific</var> is provided and true, the result is a string in the form <span class="cmd">"M.DDDDDDe+EEE"</span>, again preceded by a minus sign if and only if <var>x</var> is negative.

</article>

<article class="well bg-info text-info" id="function-sqrt">**<u>Function:</u> <span class="cmd">sqrt</span>**

sqrt -- Returns the square root of <var>x</var>.

<div class="well bg-info"><var>float</var> <span class="cmd">sqrt</span> (float <var>x</var>)</div>

Raises <span class="cmd">E_INVARG</span> if <var>x</var> is negative.

</article>

<article class="well bg-info text-info" id="function-sin">**<u>Function:</u> <span class="cmd">sin</span>**

sin -- Returns the sine of <var>x</var>.

<div class="well bg-info"><var>float</var> <span class="cmd">sin</span> (float <var>x</var>)</div>

</article>

<article class="well bg-info text-info" id="function-cos">**<u>Function:</u> <span class="cmd">cos</span>**

cos -- Returns the cosine of <var>x</var>.

<div class="well bg-info"><var>float</var> <span class="cmd">cos</span> (float <var>x</var>)</div>

</article>

<article class="well bg-info text-info" id="function-tangent">**<u>Function:</u> <span class="cmd">tangent</span>**

tangent -- Returns the tangent of <var>x</var>.

<div class="well bg-info"><var>float</var> <span class="cmd">tangent</span> (float <var>x</var>)</div>

</article>

<article class="well bg-info text-info" id="function-asin">**<u>Function:</u> <span class="cmd">asin</span>**

asin -- Returns the arc-sine (inverse sine) of <var>x</var>, in the range <span class="cmd">[-pi/2..pi/2]</span>

<div class="well bg-info"><var>float</var> <span class="cmd">asin</span> (float <var>x</var>)</div>

Raises <span class="cmd">E_INVARG</span> if <var>x</var> is outside the range <span class="cmd">[-1.0..1.0]</span>.

</article>

<article class="well bg-info text-info" id="function-acos">**<u>Function:</u> <span class="cmd">acos</span>**

acos -- Returns the arc-cosine (inverse cosine) of <var>x</var>, in the range <span class="cmd">[0..pi]</span>

<div class="well bg-info"><var>float</var> <span class="cmd">acos</span> (float <var>x</var>)</div>

Raises <span class="cmd">E_INVARG</span> if <var>x</var> is outside the range <span class="cmd">[-1.0..1.0]</span>.

</article>

<article class="well bg-info text-info" id="function-atan">**<u>Function:</u> <span class="cmd">atan</span>**

atan -- Returns the arc-tangent (inverse tangent) of <var>y</var> in the range <span class="cmd">[-pi/2..pi/2]</span>.

<div class="well bg-info"><var>float</var> <span class="cmd">atan</span> (float <var>y</var> [, float <var>x</var>])</div>

if <var>x</var> is not provided, or of <span class="cmd"><var>y</var>/<var>x</var></span> in the range <span class="cmd">[-pi..pi]</span> if <var>x</var> is provided.

</article>

<article class="well bg-info text-info" id="function-sinh">**<u>Function:</u> <span class="cmd">sinh</span>**

sinh -- Returns the hyperbolic sine of <var>x</var>.

<div class="well bg-info"><var>float</var> <span class="cmd">sinh</span> (float <var>x</var>)</div>

</article>

<article class="well bg-info text-info" id="function-cosh">**<u>Function:</u> <span class="cmd">cosh</span>**

cosh -- Returns the hyperbolic cosine of <var>x</var>.

<div class="well bg-info"><var>float</var> <span class="cmd">cosh</span> (float <var>x</var>)</div>

</article>

<article class="well bg-info text-info" id="function-tanh">**<u>Function:</u> <span class="cmd">tanh</span>**

tanh -- Returns the hyperbolic tangent of <var>x</var>.

<div class="well bg-info"><var>float</var> <span class="cmd">tanh</span> (float <var>x</var>)</div>

</article>

<article class="well bg-info text-info" id="function-exp">**<u>Function:</u> <span class="cmd">exp</span>**

exp -- Returns <var>e</var> raised to the power of <var>x</var>.

<div class="well bg-info"><var>float</var> <span class="cmd">exp</span> (float <var>x</var>)</div>

</article>

<article class="well bg-info text-info" id="function-log">**<u>Function:</u> <span class="cmd">log</span>**

log -- Returns the natural logarithm of <var>x</var>.

<div class="well bg-info"><var>float</var> <span class="cmd">log</span> (float <var>x</var>)</div>

Raises <span class="cmd">E_INVARG</span> if <var>x</var> is not positive.

</article>

<article class="well bg-info text-info" id="function-log10">**<u>Function:</u> <span class="cmd">log10</span>**

log10 -- Returns the base 10 logarithm of <var>x</var>.

<div class="well bg-info"><var>float</var> <span class="cmd">log10</span> (float <var>x</var>)</div>

Raises <span class="cmd">E_INVARG</span> if <var>x</var> is not positive.

</article>

<article class="well bg-info text-info" id="function-ceil">**<u>Function:</u> <span class="cmd">ceil</span>**

ceil -- Returns the smallest integer not less than <var>x</var>, as a floating-point number.

<div class="well bg-info"><var>float</var> <span class="cmd">ceil</span> (float <var>x</var>)</div>

</article>

<article class="well bg-info text-info" id="function-floor">**<u>Function:</u> <span class="cmd">floor</span>**

floor -- Returns the largest integer not greater than <var>x</var>, as a floating-point number.

<div class="well bg-info"><var>float</var> <span class="cmd">floor</span> (float <var>x</var>)</div>

</article>

<article class="well bg-info text-info" id="function-trunc">**<u>Function:</u> <span class="cmd">trunc</span>**

trunc -- Returns the integer obtained by truncating <var>x</var> at the decimal point, as a floating-point number.

<div class="well bg-info"><var>float</var> <span class="cmd">trunc</span> (float <var>x</var>)</div>

For negative <var>x</var>, this is equivalent to <span class="cmd">ceil()</span>; otherwise it is equivalent to <span class="cmd">floor()</span>.

</article>

##### Regular Expressions

_Regular expression_ matching allows you to test whether a string fits into a specific syntactic shape. You can also search a string for a substring that fits a pattern.

A regular expression describes a set of strings. The simplest case is one that describes a particular string; for example, the string <span class="cmd">foo</span> when regarded as a regular expression matches <span class="cmd">foo</span> and nothing else. Nontrivial regular expressions use certain special constructs so that they can match more than one string. For example, the regular expression <span class="cmd">foo%|bar</span> matches either the string <span class="cmd">foo</span> or the string <span class="cmd">bar</span>; the regular expression <span class="cmd">c[ad]*r</span> matches any of the strings <span class="cmd">cr</span>, <span class="cmd">car</span>, <span class="cmd">cdr</span>, <span class="cmd">caar</span>, <span class="cmd">cadddar</span> and all other such strings with any number of <span class="cmd">a</span>'s and <span class="cmd">d</span>'s.

Regular expressions have a syntax in which a few characters are special constructs and the rest are _ordinary_. An ordinary character is a simple regular expression that matches that character and nothing else. The special characters are <span class="cmd">$</span>, <span class="cmd">^</span>, <span class="cmd">.</span>, <span class="cmd">*</span>, <span class="cmd">+</span>, <span class="cmd">?</span>, <span class="cmd">[</span>, <span class="cmd">]</span> and <span class="cmd">%</span>. Any other character appearing in a regular expression is ordinary, unless a <span class="cmd">%</span> precedes it.

For example, <span class="cmd">f</span> is not a special character, so it is ordinary, and therefore <span class="cmd">f</span> is a regular expression that matches the string <span class="cmd">f</span> and no other string. (It does _not_, for example, match the string <span class="cmd">ff</span>.) Likewise, <span class="cmd">o</span> is a regular expression that matches only <span class="cmd">o</span>.

Any two regular expressions <var>a</var> and <var>b</var> can be concatenated. The result is a regular expression which matches a string if <var>a</var> matches some amount of the beginning of that string and <var>b</var> matches the rest of the string.

As a simple example, we can concatenate the regular expressions <span class="cmd">f</span> and <span class="cmd">o</span> to get the regular expression <span class="cmd">fo</span>, which matches only the string <span class="cmd">fo</span>. Still trivial.

The following are the characters and character sequences that have special meaning within regular expressions. Any character not mentioned here is not special; it stands for exactly itself for the purposes of searching and matching.

<dl class="dl-hortizontal">

<dt><span class="cmd">.</span></dt>

<dd>is a special character that matches any single character. Using concatenation, we can make regular expressions like <span class="cmd">a.b</span>, which matches any three-character string that begins with <span class="cmd">a</span> and ends with <span class="cmd">b</span>.</dd>

<dt><span class="cmd">*</span></dt>

<dd>is not a construct by itself; it is a suffix that means that the preceding regular expression is to be repeated as many times as possible. In <span class="cmd">fo*</span>, the <span class="cmd">*</span> applies to the <span class="cmd">o</span>, so <span class="cmd">fo*</span> matches <span class="cmd">f</span> followed by any number of <span class="cmd">o</span>'s. The case of zero <span class="cmd">o</span>'s is allowed: <span class="cmd">fo*</span> does match <span class="cmd">f</span>. <span class="cmd">*</span> always applies to the _smallest_ possible preceding expression. Thus, <span class="cmd">fo*</span> has a repeating <span class="cmd">o</span>, not a repeating <span class="cmd">fo</span>. The matcher processes a <span class="cmd">*</span> construct by matching, immediately, as many repetitions as can be found. Then it continues with the rest of the pattern. If that fails, it backtracks, discarding some of the matches of the <span class="cmd">*</span>'d construct in case that makes it possible to match the rest of the pattern. For example, matching <span class="cmd">c[ad]*ar</span> against the string <span class="cmd">caddaar</span>, the <span class="cmd">[ad]*</span> first matches <span class="cmd">addaa</span>, but this does not allow the next <span class="cmd">a</span> in the pattern to match. So the last of the matches of <span class="cmd">[ad]</span> is undone and the following <span class="cmd">a</span> is tried again. Now it succeeds.</dd>

<dt><span class="cmd">+</span></dt>

<dd><span class="cmd">+</span> is like <span class="cmd">*</span> except that at least one match for the preceding pattern is required for <span class="cmd">+</span>. Thus, <span class="cmd">c[ad]+r</span> does not match <span class="cmd">cr</span> but does match anything else that <span class="cmd">c[ad]*r</span> would match.</dd>

<dt><span class="cmd">?</span></dt>

<dd><span class="cmd">?</span> is like <span class="cmd">*</span> except that it allows either zero or one match for the preceding pattern. Thus, <span class="cmd">c[ad]?r</span> matches <span class="cmd">cr</span> or <span class="cmd">car</span> or <span class="cmd">cdr</span>, and nothing else.</dd>

<dt><span class="cmd">[ ... ]</span></dt>

<dd><span class="cmd">[</span> begins a _character set_, which is terminated by a <span class="cmd">]</span>. In the simplest case, the characters between the two brackets form the set. Thus, <span class="cmd">[ad]</span> matches either <span class="cmd">a</span> or <span class="cmd">d</span>, and <span class="cmd">[ad]*</span> matches any string of <span class="cmd">a</span>'s and <span class="cmd">d</span>'s (including the empty string), from which it follows that <span class="cmd">c[ad]*r</span> matches <span class="cmd">car</span>, etc.

Character ranges can also be included in a character set, by writing two characters with a <span class="cmd">-</span> between them. Thus, <span class="cmd">[a-z]</span> matches any lower-case letter. Ranges may be intermixed freely with individual characters, as in <span class="cmd">[a-z$%.]</span>, which matches any lower case letter or <span class="cmd">$</span>, <span class="cmd">%</span> or period.

Note that the usual special characters are not special any more inside a character set. A completely different set of special characters exists inside character sets: <span class="cmd">]</span>, <span class="cmd">-</span> and <span class="cmd">^</span>.

To include a <span class="cmd">]</span> in a character set, you must make it the first character. For example, <span class="cmd">[]a]</span> matches <span class="cmd">]</span> or <span class="cmd">a</span>. To include a <span class="cmd">-</span>, you must use it in a context where it cannot possibly indicate a range: that is, as the first character, or immediately after a range.

</dd>

<dt><span class="cmd">[^ ... ]</span></dt>

<dd>

<span class="cmd">[^</span> begins a _complement character set_, which matches any character except the ones specified. Thus, <span class="cmd">[^a-z0-9A-Z]</span> matches all characters _except_ letters and digits.

<span class="cmd">^</span> is not special in a character set unless it is the first character. The character following the <span class="cmd">^</span> is treated as if it were first (it may be a <span class="cmd">-</span> or a <span class="cmd">]</span>).

</dd>

<dt><span class="cmd">^</span></dt>

<dd>is a special character that matches the empty string -- but only if at the beginning of the string being matched. Otherwise it fails to match anything. Thus, <span class="cmd">^foo</span> matches a <span class="cmd">foo</span> which occurs at the beginning of the string.</dd>

<dt><span class="cmd">$</span></dt>

<dd>is similar to <span class="cmd">^</span> but matches only at the _end_ of the string. Thus, <span class="cmd">xx*$</span> matches a string of one or more <span class="cmd">x</span>'s at the end of the string.</dd>

<dt><span class="cmd">%</span></dt>

<dd>

has two functions: it quotes the above special characters (including <span class="cmd">%</span>), and it introduces additional special constructs.

Because <span class="cmd">%</span> quotes special characters, <span class="cmd">%$</span> is a regular expression that matches only <span class="cmd">$</span>, and <span class="cmd">%[</span> is a regular expression that matches only <span class="cmd">[</span>, and so on.

For the most part, <span class="cmd">%</span> followed by any character matches only that character. However, there are several exceptions: characters that, when preceded by <span class="cmd">%</span>, are special constructs. Such characters are always ordinary when encountered on their own.

No new special characters will ever be defined. All extensions to the regular expression syntax are made by defining new two-character constructs that begin with <span class="cmd">%</span>.

</dd>

<dt><span class="cmd">%|</span></dt>

<dd>

specifies an alternative. Two regular expressions <var>a</var> and <var>b</var> with <span class="cmd">%|</span> in between form an expression that matches anything that either <var>a</var> or <var>b</var> will match.

Thus, <span class="cmd">foo%|bar</span> matches either <span class="cmd">foo</span> or <span class="cmd">bar</span> but no other string.

<span class="cmd">%|</span> applies to the largest possible surrounding expressions. Only a surrounding <span class="cmd">%( ... %)</span> grouping can limit the grouping power of <span class="cmd">%|</span>.

Full backtracking capability exists for when multiple <span class="cmd">%|</span>'s are used.

</dd>

<dt><span class="cmd">%( ... %)</span></dt>

<dd>

is a grouping construct that serves three purposes:

1.  To enclose a set of <span class="cmd">%|</span> alternatives for other operations. Thus, <span class="cmd">%(foo%|bar%)x</span> matches either <span class="cmd">foox</span> or <span class="cmd">barx</span>.
2.  To enclose a complicated expression for a following <span class="cmd">*</span>, <span class="cmd">+</span>, or <span class="cmd">?</span> to operate on. Thus, <span class="cmd">ba%(na%)*</span> matches <span class="cmd">bananana</span>, etc., with any number of <span class="cmd">na</span>'s, including none.
3.  To mark a matched substring for future reference.

This last application is not a consequence of the idea of a parenthetical grouping; it is a separate feature that happens to be assigned as a second meaning to the same <span class="cmd">%( ... %)</span> construct because there is no conflict in practice between the two meanings. Here is an explanation of this feature:

</dd>

<dt><span class="cmd">%<var>digit</var></span></dt>

<dd>

After the end of a <span class="cmd">%( ... %)</span> construct, the matcher remembers the beginning and end of the text matched by that construct. Then, later on in the regular expression, you can use <span class="cmd">%</span> followed by <var>digit</var> to mean "match the same text matched by the <var>digit</var>'th <span class="cmd">%( ... %)</span> construct in the pattern." The <span class="cmd">%( ... %)</span> constructs are numbered in the order that their <span class="cmd">%(</span>'s appear in the pattern.

The strings matching the first nine <span class="cmd">%( ... %)</span> constructs appearing in a regular expression are assigned numbers 1 through 9 in order of their beginnings. <span class="cmd">%1</span> through <span class="cmd">%9</span> may be used to refer to the text matched by the corresponding <span class="cmd">%( ... %)</span> construct.

For example, <span class="cmd">%(.*%)%1</span> matches any string that is composed of two identical halves. The <span class="cmd">%(.*%)</span> matches the first half, which may be anything, but the <span class="cmd">%1</span> that follows must match the same exact text.

</dd>

<dt><span class="cmd">%b</span></dt>

<dd>

matches the empty string, but only if it is at the beginning or end of a word. Thus, <span class="cmd">%bfoo%b</span> matches any occurrence of <span class="cmd">foo</span> as a separate word. <span class="cmd">%bball%(s%|%)%b</span> matches <span class="cmd">ball</span> or <span class="cmd">balls</span> as a separate word.

For the purposes of this construct and the five that follow, a word is defined to be a sequence of letters and/or digits.

</dd>

<dt><span class="cmd">%B</span></dt>

<dd>matches the empty string, provided it is _not_ at the beginning or end of a word.</dd>

<dt><span class="cmd">%<</span></dt>

<dd>matches the empty string, but only if it is at the beginning of a word.</dd>

<dt><span class="cmd">%></span></dt>

<dd>matches the empty string, but only if it is at the end of a word.</dd>

<dt><span class="cmd">%w</span></dt>

<dd>matches any word-constituent character (i.e., any letter or digit).</dd>

<dt><span class="cmd">%W</span></dt>

<dd>matches any character that is not a word constituent.</dd>

</dl>

##### Operations on Strings

<article class="well bg-info text-info" id="function-length">**<u>Function:</u> <span class="cmd">length</span>**

length -- Returns the number of characters in <var>string</var>.

<div class="well bg-info"><var>int</var> <span class="cmd">length</span> (str <var>string</var>)</div>

It is also permissible to pass a list to <span class="cmd">length()</span>; see the description in the next section.

<pre>length("foo")   =>   3
length("")      =>   0
</pre>

</article>

<article class="well bg-info text-info" id="function-strsub">**<u>Function:</u> <span class="cmd">strsub</span>**

strsub -- Replaces all occurrences of <var>what</var> in <var>subject</var> with <var>with</var>, performing string substitution.

<div class="well bg-info"><var>str</var> <span class="cmd">strsub</span> (str <var>subject</var>, str <var>what</var>, str <var>with</var> [, int <var>case-matters</var>])</div>

The occurrences are found from left to right and all substitutions happen simultaneously. By default, occurrences of <var>what</var> are searched for while ignoring the upper/lower case distinction. If <var>case-matters</var> is provided and true, then case is treated as significant in all comparisons.

<pre>strsub("%n is a fink.", "%n", "Fred")   =>   "Fred is a fink."
strsub("foobar", "OB", "b")             =>   "fobar"
strsub("foobar", "OB", "b", 1)          =>   "foobar"
</pre>

</article>

<article class="well bg-info text-info" id="function-index">**<u>Function:</u> <span class="cmd">index</span>**

index -- Returns the index of the first character of the first occurrence of <var>str2</var> in <var>str1</var>.

<div class="well bg-info"><var>int</var> <span class="cmd">index</span> (str <var>str1</var>, str <var>str2</var>, [, int <var>case-matters</var>])</div>

If <var>str2</var> does not occur in <var>str1</var> at all, zero is returned. By default the search for an occurrence of <var>str2</var> is done while ignoring the upper/lower case distinction. If <var>case-matters</var> is provided and true, then case is treated as significant in all comparisons.

<pre>index("foobar", "o")        =>   2
index("foobar", "x")        =>   0
index("foobar", "oba")      =>   3
index("Foobar", "foo", 1)   =>   0
</pre>

</article>

<article class="well bg-info text-info" id="function-rindex">**<u>Function:</u> <span class="cmd">rindex</span>**

rindex -- Returns the index of the first character of the last occurrence of <var>str2</var> in <var>str1</var>.

<div class="well bg-info"><var>int</var> <span class="cmd">rindex</span> (str <var>str1</var>, str <var>str2</var>, [, int <var>case-matters</var>])</div>

If <var>str2</var> does not occur in <var>str1</var> at all, zero is returned. By default the search for an occurrence of <var>str2</var> is done while ignoring the upper/lower case distinction. If <var>case-matters</var> is provided and true, then case is treated as significant in all comparisons.

<pre>rindex("foobar", "o")       =>   3
</pre>

</article>

<article class="well bg-info text-info" id="function-strcmp">**<u>Function:</u> <span class="cmd">strcmp</span>**

strcmp -- Performs a case-sensitive comparison of the two argument strings.

<div class="well bg-info"><var>int</var> <span class="cmd">strcmp</span> (str <var>str1</var>, str <var>str2</var>)</div>

If <var>str1</var> is [lexicographically](https://en.wikipedia.org/wiki/Lexicographical_order) less than <var>str2</var>, the <span class="cmd">strcmp()</span> returns a negative integer. If the two strings are identical, <span class="cmd">strcmp()</span> returns zero. Otherwise, <span class="cmd">strcmp()</span> returns a positive integer. The ASCII character ordering is used for the comparison.

</article>

<article class="well bg-info text-info" id="function-decode-binary">**<u>Function:</u> <span class="cmd">decode_binary</span>**

decode_binary -- Returns a list of strings and/or integers representing the bytes in the binary string <var>bin_string</var> in order.

<div class="well bg-info"><var>list</var> <span class="cmd">decode_binary</span> (str <var>bin-string</var> [, int <var>fully</var>])</div>

If <var>fully</var> is false or omitted, the list contains an integer only for each non-printing, non-space byte; all other characters are grouped into the longest possible contiguous substrings. If <var>fully</var> is provided and true, the list contains only integers, one for each byte represented in <var>bin_string</var>. Raises <span class="cmd">E_INVARG</span> if <var>bin_string</var> is not a properly-formed binary string. (See the early section on MOO value types for a full description of binary strings.)

<pre>decode_binary("foo")               =>   {"foo"}
decode_binary("~~foo")             =>   {"~foo"}
decode_binary("foo~0D~0A")         =>   {"foo", 13, 10}
decode_binary("foo~0Abar~0Abaz")   =>   {"foo", 10, "bar", 10, "baz"}
decode_binary("foo~0D~0A", 1)      =>   {102, 111, 111, 13, 10}
</pre>

</article>

<article class="well bg-info text-info" id="function-encode-binary">**<u>Function:</u> <span class="cmd">encode_binary</span>**

encode_binary -- Translates each integer and string in turn into its binary string equivalent, returning the concatenation of all these substrings into a single binary string.

<div class="well bg-info"><var>str</var> <span class="cmd">encode_binary</span> (<var>arg</var>, ...)</div>

Each argument must be an integer between 0 and 255, a string, or a list containing only legal arguments for this function. This function (See the early section on MOO value types for a full description of binary strings.)

<pre>encode_binary("~foo")                     =>   "~7Efoo"
encode_binary({"foo", 10}, {"bar", 13})   =>   "foo~0Abar~0D"
encode_binary("foo", 10, "bar", 13)       =>   "foo~0Abar~0D"
</pre>

</article>

<article class="well bg-info text-info" id="function-match">**<u>Function:</u> <span class="cmd">match</span>**

match -- Searches for the first occurrence of the regular expression <var>pattern</var> in the string <var>subject</var>

<div class="well bg-info"><var>list</var> <span class="cmd">match</span> (str <var>subject</var>, str <var>pattern</var> [, int <var>case-matters</var>])</div>

If <var>pattern</var> is syntactically malformed, then <span class="cmd">E_INVARG</span> is raised. The process of matching can in some cases consume a great deal of memory in the server; should this memory consumption become excessive, then the matching process is aborted and <span class="cmd">E_QUOTA</span> is raised.

If no match is found, the empty list is returned; otherwise, these functions return a list containing information about the match (see below). By default, the search ignores upper-/lower-case distinctions. If <var>case-matters</var> is provided and true, then case is treated as significant in all comparisons.

The list that <span class="cmd">match()</span> returns contains the details about the match made. The list is in the form:

<pre>{<var>start</var>, <var>end</var>, <var>replacements</var>, <var>subject</var>}
</pre>

where <var>start</var> is the index in <var>subject</var> of the beginning of the match, <var>end</var> is the index of the end of the match, <var>replacements</var> is a list described below, and <var>subject</var> is the same string that was given as the first argument to <span class="cmd">match()</span>.

The <var>replacements</var> list is always nine items long, each item itself being a list of two integers, the start and end indices in <var>string</var> matched by some parenthesized sub-pattern of <var>pattern</var>. The first item in <var>replacements</var> carries the indices for the first parenthesized sub-pattern, the second item carries those for the second sub-pattern, and so on. If there are fewer than nine parenthesized sub-patterns in <var>pattern</var>, or if some sub-pattern was not used in the match, then the corresponding item in <var>replacements</var> is the list {0, -1}. See the discussion of <span class="cmd">%)</span>, below, for more information on parenthesized sub-patterns.

<pre>match("foo", "^f*o$")        =>  {}
match("foo", "^fo*$")        =>  {1, 3, {{0, -1}, ...}, "foo"}
match("foobar", "o*b")       =>  {2, 4, {{0, -1}, ...}, "foobar"}
match("foobar", "f%(o*%)b")
        =>  {1, 4, {{2, 3}, {0, -1}, ...}, "foobar"}
</pre>

</article>

<article class="well bg-info text-info" id="function-rmatch">**<u>Function:</u> <span class="cmd">rmatch</span>**

rmatch -- Searches for the last occurrence of the regular expression <var>pattern</var> in the string <var>subject</var>

2

<div class="well bg-info"><var>list</var> <span class="cmd">rmatch</span> (str <var>subject</var>, str <var>pattern</var> [, int <var>case-matters</var>])</div>

If <var>pattern</var> is syntactically malformed, then <span class="cmd">E_INVARG</span> is raised. The process of matching can in some cases consume a great deal of memory in the server; should this memory consumption become excessive, then the matching process is aborted and <span class="cmd">E_QUOTA</span> is raised.

If no match is found, the empty list is returned; otherwise, these functions return a list containing information about the match (see below). By default, the search ignores upper-/lower-case distinctions. If <var>case-matters</var> is provided and true, then case is treated as significant in all comparisons.

The list that <span class="cmd">match()</span> returns contains the details about the match made. The list is in the form:

<pre>{<var>start</var>, <var>end</var>, <var>replacements</var>, <var>subject</var>}
</pre>

where <var>start</var> is the index in <var>subject</var> of the beginning of the match, <var>end</var> is the index of the end of the match, <var>replacements</var> is a list described below, and <var>subject</var> is the same string that was given as the first argument to <span class="cmd">match()</span>.

The <var>replacements</var> list is always nine items long, each item itself being a list of two integers, the start and end indices in <var>string</var> matched by some parenthesized sub-pattern of <var>pattern</var>. The first item in <var>replacements</var> carries the indices for the first parenthesized sub-pattern, the second item carries those for the second sub-pattern, and so on. If there are fewer than nine parenthesized sub-patterns in <var>pattern</var>, or if some sub-pattern was not used in the match, then the corresponding item in <var>replacements</var> is the list {0, -1}. See the discussion of <span class="cmd">%)</span>, below, for more information on parenthesized sub-patterns.

<pre>rmatch("foobar", "o*b")      =>  {4, 4, {{0, -1}, ...}, "foobar"}
</pre>

</article>

<article class="well bg-info text-info" id="function-substitute">**<u>Function:</u> <span class="cmd">substitute</span>**

substitute -- Performs a standard set of substitutions on the string <var>template</var>, using the information contained in <var>subs</var>, returning the resulting, transformed <var>template</var>.

<div class="well bg-info"><var>str</var> <span class="cmd">substitute</span> (str <var>template</var>, list <var>subs</var>)</div>

<var>Subs</var> should be a list like those returned by <span class="cmd">match()</span> or <span class="cmd">rmatch()</span> when the match succeeds; otherwise, <span class="cmd">E_INVARG</span> is raised.

In <var>template</var>, the strings <span class="cmd">%1</span> through <span class="cmd">%9</span> will be replaced by the text matched by the first through ninth parenthesized sub-patterns when <span class="cmd">match()</span> or <span class="cmd">rmatch()</span> was called. The string <span class="cmd">%0</span> in <var>template</var> will be replaced by the text matched by the pattern as a whole when <span class="cmd">match()</span> or <span class="cmd">rmatch()</span> was called. The string <span class="cmd">%%</span> will be replaced by a single <span class="cmd">%</span> sign. If <span class="cmd">%</span> appears in <var>template</var> followed by any other character, <span class="cmd">E_INVARG</span> will be raised.

<pre>subs = match("*** Welcome to LambdaMOO!!!", "%(%w*%) to %(%w*%)");
substitute("I thank you for your %1 here in %2.", subs)
        =>   "I thank you for your Welcome here in LambdaMOO."
</pre>

</article>

<article class="well bg-info text-info" id="function-crypt">**<u>Function:</u> <span class="cmd">crypt</span>**

crypt -- Encrypts the given <var>text</var> using the standard UNIX encryption method.

<div class="well bg-info"><var>str</var> <span class="cmd">crypt</span> (str <var>text</var> [, str <var>salt</var>])</div>

If provided, <var>salt</var> should be a string at least two characters long, the first two characters of which will be used as the extra encryption "salt" in the algorithm. If <var>salt</var> is not provided, a random pair of characters is used. In any case, the salt used is also returned as the first two characters of the resulting encrypted string.

Aside from the possibly-random selection of the salt, the encryption algorithm is entirely deterministic. In particular, you can test whether or not a given string is the same as the one used to produce a given piece of encrypted text; simply extract the first two characters of the encrypted text and pass the candidate string and those two characters to <span class="cmd">crypt()</span>. If the result is identical to the given encrypted text, then you've got a match.

<pre>crypt("foobar")         =>   "J3fSFQfgkp26w"
crypt("foobar", "J3")   =>   "J3fSFQfgkp26w"
crypt("mumble", "J3")   =>   "J3D0.dh.jjmWQ"
crypt("foobar", "J4")   =>   "J4AcPxOJ4ncq2"
</pre>

</article>

<article class="well bg-info text-info" id="function-string_hash">**<u>Function:</u> <span class="cmd">string_hash</span>**  
**<u>Function:</u> <span class="cmd">binary_hash</span>**

string_hash -- Returns a 32-character hexadecimal string.

binary_hash -- Returns a 32-character hexadecimal string.

<div class="well bg-info"><var>str</var> <span class="cmd">string_hash</span> (str <var>string</var>)</div>

<div class="well bg-info"><var>str</var> <span class="cmd">binary_hash</span> (str <var>bin-string</var>)</div>

Returns the result of applying the MD5 cryptographically secure hash function to the contents of the string <var>string</var> or the binary string <var>bin-string</var>. MD5, like other such functions, has the property that, if

<pre>string_hash(<var>x</var>) == string_hash(<var>y</var>)
</pre>

then, almost certainly,

<pre>equal(<var>x</var>, <var>y</var>)
</pre>

This can be useful, for example, in certain networking applications: after sending a large piece of text across a connection, also send the result of applying <span class="cmd">string_hash()</span> to the text; if the destination site also applies <span class="cmd">string_hash()</span> to the text and gets the same result, you can be quite confident that the large text has arrived unchanged.

</article>

##### Operations on Lists

<article class="well bg-info text-info" id="function-length-list">**<u>Function:</u> <span class="cmd">length</span>**

length -- Returns the number of elements in <var>list</var>.

<div class="well bg-info"><var>int</var> <span class="cmd">length</span> (list <var>list</var>)</div>

It is also permissible to pass a string to <span class="cmd">length()</span>; see the description in the previous section.

<pre>length({1, 2, 3})   =>   3
length({})          =>   0
</pre>

</article>

<article class="well bg-info text-info" id="function-is-member">**<u>Function:</u> <span class="cmd">is_member</span>**

is_member -- Returns true if there is an element of <var>list</var> that is completely indistinguishable from <var>value</var>.

<div class="well bg-info"><var>int</var> <span class="cmd">is_member</span> (<var>value</var>, list <var>list</var>)</div>

This is much the same operation as "<span class="cmd"><var>value</var> in <var>list</var></span>" except that, unlike <span class="cmd">in</span>, the <span class="cmd">is_member()</span> function does not treat upper- and lower-case characters in strings as equal.

<pre>"Foo" in {1, "foo", #24}            =>   2
is_member("Foo", {1, "foo", #24})   =>   0
is_member("Foo", {1, "Foo", #24})   =>   2
</pre>

</article>

<article class="well bg-info text-info" id="function-listinsert-listdelete">**<u>Function:</u> <span class="cmd">listinsert</span>**  
**<u>Function:</u> <span class="cmd">listappend</span>**

listinsert -- This functions return a copy of <var>list</var> with <var>value</var> added as a new element.

listappend -- This functions return a copy of <var>list</var> with <var>value</var> added as a new element.

<div class="well bg-info"><var>list</var> <span class="cmd">listinsert</span> (list <var>list</var>, <var>value</var> [, int <var>index</var>])</div>

<div class="well bg-info"><var>list</var> <span class="cmd">listappend</span> (list <var>list</var>, <var>value</var> [, int <var>index</var>])</div>

<span class="cmd">listinsert()</span> and <span class="cmd">listappend()</span> add <var>value</var> before and after (respectively) the existing element with the given <var>index</var>, if provided.

The following three expressions always have the same value:

<pre>listinsert(<var>list</var>, <var>element</var>, <var>index</var>)
listappend(<var>list</var>, <var>element</var>, <var>index</var> - 1)
{@<var>list</var>[1..<var>index</var> - 1], <var>element</var>, @<var>list</var>[<var>index</var>..length(<var>list</var>)]}
</pre>

If <var>index</var> is not provided, then <span class="cmd">listappend()</span> adds the <var>value</var> at the end of the list and <span class="cmd">listinsert()</span> adds it at the beginning; this usage is discouraged, however, since the same intent can be more clearly expressed using the list-construction expression, as shown in the examples below.

<pre>x = {1, 2, 3};
listappend(x, 4, 2)   =>   {1, 2, 4, 3}
listinsert(x, 4, 2)   =>   {1, 4, 2, 3}
listappend(x, 4)      =>   {1, 2, 3, 4}
listinsert(x, 4)      =>   {4, 1, 2, 3}
{@x, 4}               =>   {1, 2, 3, 4}
{4, @x}               =>   {4, 1, 2, 3}
</pre>

</article>

<article class="well bg-info text-info" id="function-listdelete">**<u>Function:</u> <span class="cmd">listdelete</span>**

listdelete -- Returns a copy of <var>list</var> with the <var>index</var>th element removed.

<div class="well bg-info"><var>list</var> <span class="cmd">listdelete</span> (list <var>list</var>, int <var>index</var>)</div>

If <var>index</var> is not in the range <span class="cmd">[1..length(<var>list</var>)]</span>, then <span class="cmd">E_RANGE</span> is raised.

<pre>x = {"foo", "bar", "baz"};
listdelete(x, 2)   =>   {"foo", "baz"}
</pre>

</article>

<article class="well bg-info text-info" id="function-is-listset">**<u>Function:</u> <span class="cmd">listset</span>**

listset -- Returns a copy of <var>list</var> with the <var>index</var>th element replaced by <var>value</var>.

<div class="well bg-info"><var>list</var> <span class="cmd">listset</span> (list <var>list</var>, <var>value</var>, int <var>index</var>)</div>

If <var>index</var> is not in the range <span class="cmd">[1..length(<var>list</var>)]</span>, then <span class="cmd">E_RANGE</span> is raised.

<pre>x = {"foo", "bar", "baz"};
listset(x, "mumble", 2)   =>   {"foo", "mumble", "baz"}
</pre>

This function exists primarily for historical reasons; it was used heavily before the server supported indexed assignments like <span class="cmd">x[i] = v</span>. New code should always use indexed assignment instead of <span class="cmd">listset()</span> wherever possible.

</article>

<article class="well bg-info text-info" id="function-setadd-setremote">**<u>Function:</u> <span class="cmd">setadd</span>**  
**<u>Function:</u> <span class="cmd">setremove</span>**

setadd -- Returns a copy of <var>list</var> with the given <var>value</var> added.

setremove -- Returns a copy of <var>list</var> with the given <var>value</var> removed.

<div class="well bg-info"><var>list</var> <span class="cmd">setadd</span> (list <var>list</var>, <var>value</var>)</div>

<div class="well bg-info"><var>list</var> <span class="cmd">setremove</span> (list <var>list</var>, <var>value</var>)</div>

<span class="cmd">setadd()</span> only adds <var>value</var> if it is not already an element of <var>list</var>; <var>list</var> is thus treated as a mathematical set. <var>value</var> is added at the end of the resulting list, if at all. Similarly, <span class="cmd">setremove()</span> returns a list identical to <var>list</var> if <var>value</var> is not an element. If <var>value</var> appears more than once in <var>list</var>, only the first occurrence is removed in the returned copy.

<pre>setadd({1, 2, 3}, 3)         =>   {1, 2, 3}
setadd({1, 2, 3}, 4)         =>   {1, 2, 3, 4}
setremove({1, 2, 3}, 3)      =>   {1, 2}
setremove({1, 2, 3}, 4)      =>   {1, 2, 3}
setremove({1, 2, 3, 2}, 2)   =>   {1, 3, 2}
</pre>

</article>

#### Manipulating Objects

Objects are, of course, the main focus of most MOO programming and, largely due to that, there are a lot of built-in functions for manipulating them.

##### Fundamental Operations on Objects

<article class="well bg-info text-info" id="function-create">**<u>Function:</u> <span class="cmd">create</span>**

create -- Creates and returns a new object whose parent is <var>parent</var> and whose owner is as described below.

<div class="well bg-info"><var>obj</var> <span class="cmd">create</span> (obj <var>parent</var> [, obj <var>owner</var>])</div>

Either the given <var>parent</var> object must be <span class="cmd">#-1</span> or valid and fertile (i.e., its <span class="cmd">f</span> bit must be set) or else the programmer must own <var>parent</var> or be a wizard; otherwise <span class="cmd">E_PERM</span> is raised. <span class="cmd">E_PERM</span> is also raised if <var>owner</var> is provided and not the same as the programmer, unless the programmer is a wizard. After the new object is created, its <span class="cmd">initialize</span> verb, if any, is called with no arguments.

The new object is assigned the least non-negative object number that has not yet been used for a created object. Note that no object number is ever reused, even if the object with that number is recycled.

<div class="alert alert-warning">This is not strictly true, especially if you are using LambdaCore and the <span class="cmd">$recycler</span>, which is a great idea. If you don't, you end up with extremely high object numbers. However, if you plan on reusing object numbers you need to consider this carefully in your code. You do not want to include object numbers in your code if this is the case, as object numbers could change. Use corified references instead (IE: <span class="cmd">@prop #0.my_object #objnum</span> allows you to use $my_object in your code. If the object number ever changes, you can change the reference without updating all of your code.)</div>

The owner of the new object is either the programmer (if <var>owner</var> is not provided), the new object itself (if <var>owner</var> was given as <span class="cmd">#-1</span>), or <var>owner</var> (otherwise).

The other built-in properties of the new object are initialized as follows:

<pre>name         ""
location     #-1
contents     {}
programmer   0
wizard       0
r            0
w            0
f            0
</pre>

The function <span class="cmd">is_player()</span> returns false for newly created objects.

In addition, the new object inherits all of the other properties on <var>parent</var>. These properties have the same permission bits as on <var>parent</var>. If the <span class="cmd">c</span> permissions bit is set, then the owner of the property on the new object is the same as the owner of the new object itself; otherwise, the owner of the property on the new object is the same as that on <var>parent</var>. The initial value of every inherited property is _clear_; see the description of the built-in function <span class="cmd">clear_property()</span> for details.

If the intended owner of the new object has a property named <span class="cmd">ownership_quota</span> and the value of that property is an integer, then <span class="cmd">create()</span> treats that value as a _quota_. If the quota is less than or equal to zero, then the quota is considered to be exhausted and <span class="cmd">create()</span> raises <span class="cmd">E_QUOTA</span> instead of creating an object. Otherwise, the quota is decremented and stored back into the <span class="cmd">ownership_quota</span> property as a part of the creation of the new object.

</article>

<article class="well bg-info text-info" id="function-chparent">**<u>Function:</u> <span class="cmd">chparent</span>**

chparent -- Changes the parent of <var>object</var> to be <var>new-parent</var>.

<div class="well bg-info"><var>none</var> <span class="cmd">chparent</span> (obj <var>object</var>, obj <var>new-parent</var>)</div>

If <var>object</var> is not valid, or if <var>new-parent</var> is neither valid nor equal to <span class="cmd">#-1</span>, then <span class="cmd">E_INVARG</span> is raised. If the programmer is neither a wizard or the owner of <var>object</var>, or if <var>new-parent</var> is not fertile (i.e., its <span class="cmd">f</span> bit is not set) and the programmer is neither the owner of <var>new-parent</var> nor a wizard, then <span class="cmd">E_PERM</span> is raised. If <var>new-parent</var> is equal to <span class="cmd">object</span> or one of its current ancestors, <span class="cmd">E_RECMOVE</span> is raised. If <var>object</var> or one of its descendants defines a property with the same name as one defined either on <var>new-parent</var> or on one of its ancestors, then <span class="cmd">E_INVARG</span> is raised.

Changing an object's parent can have the effect of removing some properties from and adding some other properties to that object and all of its descendants (i.e., its children and its children's children, etc.). Let <var>common</var> be the nearest ancestor that <var>object</var> and <var>new-parent</var> have in common before the parent of <var>object</var> is changed. Then all properties defined by ancestors of <var>object</var> under <var>common</var> (that is, those ancestors of <var>object</var> that are in turn descendants of <var>common</var>) are removed from <var>object</var> and all of its descendants. All properties defined by <var>new-parent</var> or its ancestors under <var>common</var> are added to <var>object</var> and all of its descendants. As with <span class="cmd">create()</span>, the newly-added properties are given the same permission bits as they have on <var>new-parent</var>, the owner of each added property is either the owner of the object it's added to (if the <span class="cmd">c</span> permissions bit is set) or the owner of that property on <var>new-parent</var>, and the value of each added property is _clear_; see the description of the built-in function <span class="cmd">clear_property()</span> for details. All properties that are not removed or added in the reparenting process are completely unchanged.

If <var>new-parent</var> is equal to <span class="cmd">#-1</span>, then <var>object</var> is given no parent at all; it becomes a new root of the parent/child hierarchy. In this case, all formerly inherited properties on <var>object</var> are simply removed.

</article>

<article class="well bg-info text-info" id="function-valid">**<u>Function:</u> <span class="cmd">valid</span>**

valid -- Return a non-zero integer if object is valid and not yet recycled.

<div class="well bg-info"><var>int</var> <span class="cmd">valid</span> (obj <var>object</var>)</div>

Returns a non-zero integer (i.e., a true value) if <var>object</var> is a valid object (one that has been created and not yet recycled) and zero (i.e., a false value) otherwise.

<pre>valid(#0)    =>   1
valid(#-1)   =>   0
</pre>

</article>

<article class="well bg-info text-info" id="function-parent">**<u>Function:</u> <span class="cmd">parent</span>**

parent -- return the parent of <var>object</var>

<div class="well bg-info"><var>obj</var> <span class="cmd">parent</span> (obj <var>object</var>)</div>

</article>

<article class="well bg-info text-info" id="function-children">**<u>Function:</u> <span class="cmd">children</span>**

children -- return a list of the children of <var>object</var>.

<div class="well bg-info"><var>list</var> <span class="cmd">children</span> (obj <var>object</var>)</div>

</article>

<article class="well bg-info text-info" id="function-recycle">**<u>Function:</u> <span class="cmd">recycle</span>**

recycle -- destroy <var>object</var> irrevocably.

<div class="well bg-info"><var>none</var> <span class="cmd">recycle</span> (obj <var>object</var>)</div>

The given <var>object</var> is destroyed, irrevocably. The programmer must either own <var>object</var> or be a wizard; otherwise, <span class="cmd">E_PERM</span> is raised. If <var>object</var> is not valid, then <span class="cmd">E_INVARG</span> is raised. The children of <var>object</var> are reparented to the parent of <var>object</var>. Before <var>object</var> is recycled, each object in its contents is moved to <span class="cmd">#-1</span> (implying a call to <var>object</var>'s <span class="cmd">exitfunc</span> verb, if any) and then <var>object</var>'s <span class="cmd">recycle</span> verb, if any, is called with no arguments.

After <var>object</var> is recycled, if the owner of the former object has a property named <span class="cmd">ownership_quota</span> and the value of that property is a integer, then <span class="cmd">recycle()</span> treats that value as a _quota_ and increments it by one, storing the result back into the <span class="cmd">ownership_quota</span> property.>

</article>

<article class="well bg-info text-info" id="function-object-bytes">**<u>Function:</u> <span class="cmd">object_bytes</span>**

object_bytes -- Returns the number of bytes of the server's memory required to store the given <var>object</var>.

<div class="well bg-info"><var>int</var> <span class="cmd">object_bytes</span> (obj <var>object</var>)</div>

The space calculation includes the space used by the values of all of the objects non-clear properties and by the verbs and properties defined directly on the object.

Raises <span class="cmd">E_INVARG</span> if <var>object</var> is not a valid object and <span class="cmd">E_PERM</span> if the programmer is not a wizard.

</article>

<article class="well bg-info text-info" id="function-max-object">**<u>Function:</u> <span class="cmd">max_object</span>**

max_object -- Returns the largest object number ever assigned to a created object.

<div class="well bg-info"><var>obj</var> <span class="cmd">max_object</span> ()</div>

Note that the object with this number may no longer exist; it may have been recycled. The next object created will be assigned the object number one larger than the value of <span class="cmd">max_object()</span>.

<div class="alert alert-warning">The next object getting the number one larger than <span class="cmd">max_object()</span> only applies if you are using built-in functions for creating objects and does not apply if you are using the <span class="cmd">$recycler</span> to create objects.</div>

</article>

##### Object Movement

<article class="well bg-info text-info" id="function-move">**<u>Function:</u> <span class="cmd">move</span>**

move -- Changes <var>what</var>'s location to be <var>where</var>.

<div class="well bg-info"><var>none</var> <span class="cmd">move</span> (obj <var>what</var>, obj <var>where</var>)</div>

This is a complex process because a number of permissions checks and notifications must be performed. The actual movement takes place as described in the following paragraphs.

<var>what</var> should be a valid object and <var>where</var> should be either a valid object or <span class="cmd">#-1</span> (denoting a location of `nowhere'); otherwise <span class="cmd">E_INVARG</span> is raised. The programmer must be either the owner of <var>what</var> or a wizard; otherwise, <span class="cmd">E_PERM</span> is raised.

If <var>where</var> is a valid object, then the verb-call

<pre><var>where</var>:accept(<var>what</var>)
</pre>

is performed before any movement takes place. If the verb returns a false value and the programmer is not a wizard, then <var>where</var> is considered to have refused entrance to <var>what</var>; <span class="cmd">move()</span> raises <span class="cmd">E_NACC</span>. If <var>where</var> does not define an <span class="cmd">accept</span> verb, then it is treated as if it defined one that always returned false.

If moving <var>what</var> into <var>where</var> would create a loop in the containment hierarchy (i.e., <var>what</var> would contain itself, even indirectly), then <span class="cmd">E_RECMOVE</span> is raised instead.

The <span class="cmd">location</span> property of <var>what</var> is changed to be <var>where</var>, and the <span class="cmd">contents</span> properties of the old and new locations are modified appropriately. Let <var>old-where</var> be the location of <var>what</var> before it was moved. If <var>old-where</var> is a valid object, then the verb-call

<pre><var>old-where</var>:exitfunc(<var>what</var>)
</pre>

is performed and its result is ignored; it is not an error if <var>old-where</var> does not define a verb named <span class="cmd">exitfunc</span>. Finally, if <var>where</var> and <var>what</var> are still valid objects, and <var>where</var> is still the location of <var>what</var>, then the verb-call

<pre><var>where</var>:enterfunc(<var>what</var>)
</pre>

is performed and its result is ignored; again, it is not an error if <var>where</var> does not define a verb named <span class="cmd">enterfunc</span>.

</article>

##### Operations on Properties

<article class="well bg-info text-info" id="function-properties">**<u>Function:</u> <span class="cmd">properties</span>**

properties -- Returns a list of the names of the properties defined directly on the given <var>object</var>, not inherited from its parent.

<div class="well bg-info"><var>list</var> <span class="cmd">properties</span> (obj <var>object</var>)</div>

If <var>object</var> is not valid, then <span class="cmd">E_INVARG</span> is raised. If the programmer does not have read permission on <var>object</var>, then <span class="cmd">E_PERM</span> is raised.

</article>

<article class="well bg-info text-info" id="function-property-info">**<u>Function:</u> <span class="cmd">property_info</span>**

property_info -- Get the owner and permission bits for the property named prop-name on the given object

<div class="well bg-info"><var>list</var> <span class="cmd">property_info (obj <var>object</var>, str <var>prop-name</var>)</span></div>

If <var>object</var> is not valid, then <span class="cmd">E_INVARG</span> is raised. If <var>object</var> has no non-built-in property named <var>prop-name</var>, then <span class="cmd">E_PROPNF</span> is raised. If the programmer does not have read (write) permission on the property in question, then <span class="cmd">property_info()</span> raises <span class="cmd">E_PERM</span>.

</article>

<article class="well bg-info text-info" id="function-set-property-info">**<u>Function:</u> <span class="cmd">set_property_info</span>**

set_property_info -- Set the owner and permission bits for the property named prop-name on the given object

<div class="well bg-info"><var>none</var> <span class="cmd">set_property_info</span> (obj <var>object</var>, str <var>prop-name</var>, list <var>info</var>)</div>

If <var>object</var> is not valid, then <span class="cmd">E_INVARG</span> is raised. If <var>object</var> has no non-built-in property named <var>prop-name</var>, then <span class="cmd">E_PROPNF</span> is raised. If the programmer does not have read (write) permission on the property in question, then <span class="cmd">set_property_info()</span> raises <span class="cmd">E_PERM</span>. Property info has the following form:

<pre>{<var>owner</var>, <var>perms</var> [, <var>new-name</var>]}
</pre>

where <var>owner</var> is an object, <var>perms</var> is a string containing only characters from the set <span class="cmd">r</span>, <span class="cmd">w</span>, and <span class="cmd">c</span>, and <var>new-name</var> is a string; <var>new-name</var> is never part of the value returned by <span class="cmd">property_info()</span>, but it may optionally be given as part of the value provided to <span class="cmd">set_property_info()</span>. This list is the kind of value returned by <span class="cmd">property_info()</span> and expected as the third argument to <span class="cmd">set_property_info()</span>; the latter function raises <span class="cmd">E_INVARG</span> if <var>owner</var> is not valid, if <var>perms</var> contains any illegal characters, or, when <var>new-name</var> is given, if <var>prop-name</var> is not defined directly on <var>object</var> or <var>new-name</var> names an existing property defined on <var>object</var> or any of its ancestors or descendants.

</article>

<article class="well bg-info text-info" id="function-add-property">**<u>Function:</u> <span class="cmd">add_property</span>**

add_property -- Defines a new property on the given <var>object</var>

<div class="well bg-info"><var>none</var> <span class="cmd">add_property</span> (obj <var>object</var>, str <var>prop-name</var>, <var>value</var>, list <var>info</var>)</div>

The property is inherited by all of its descendants; the property is named <var>prop-name</var>, its initial value is <var>value</var>, and its owner and initial permission bits are given by <var>info</var> in the same format as is returned by <span class="cmd">property_info()</span>, described above.

If <var>object</var> is not valid or <var>info</var> does not specify a valid owner and well-formed permission bits or <var>object</var> or its ancestors or descendants already defines a property named <var>prop-name</var>, then <span class="cmd">E_INVARG</span> is raised. If the programmer does not have write permission on <var>object</var> or if the owner specified by <var>info</var> is not the programmer and the programmer is not a wizard, then <span class="cmd">E_PERM</span> is raised.

</article>

<article class="well bg-info text-info" id="function-delete-property">**<u>Function:</u> <span class="cmd">delete_property</span>**

delete_property -- Removes the property named <var>prop-name</var> from the given <var>object</var> and all of its descendants.

<div class="well bg-info"><var>none</var> <span class="cmd">>delete_property</span> (obj <var>object</var>, str <var>prop-name</var>)</div>

If <var>object</var> is not valid, then <span class="cmd">E_INVARG</span> is raised. If the programmer does not have write permission on <var>object</var>, then <span class="cmd">E_PERM</span> is raised. If <var>object</var> does not directly define a property named <var>prop-name</var> (as opposed to inheriting one from its parent), then <span class="cmd">E_PROPNF</span> is raised.

</article>

<article class="well bg-info text-info" id="function-is-clear-property">**<u>Function:</u> <span class="cmd">is_clear_property</span>**

is_clear_property -- Test the specified property for clear

<div class="well bg-info"><var>int</var> <span class="cmd">is_clear_property</span> (obj <var>object</var>, str <var>prop-name</var>)</div>

**<u>Function:</u> <span class="cmd">clear_property</span>**

clear_property -- Set the specified property to clear

<div class="well bg-info"><var>none</var> <span class="cmd">clear_property</span> (obj <var>object</var>, str <var>prop-name</var>)</div>

These two functions test for clear and set to clear, respectively, the property named <var>prop-name</var> on the given <var>object</var>. If <var>object</var> is not valid, then <span class="cmd">E_INVARG</span> is raised. If <var>object</var> has no non-built-in property named <var>prop-name</var>, then <span class="cmd">E_PROPNF</span> is raised. If the programmer does not have read (write) permission on the property in question, then <span class="cmd">is_clear_property()</span> (<span class="cmd">clear_property()</span>) raises <span class="cmd">E_PERM</span>.

If a property is clear, then when the value of that property is queried the value of the parent's property of the same name is returned. If the parent's property is clear, then the parent's parent's value is examined, and so on. If <var>object</var> is the definer of the property <var>prop-name</var>, as opposed to an inheritor of the property, then <span class="cmd">clear_property()</span> raises <span class="cmd">E_INVARG</span>.

</article>

##### Operations on Verbs

<article class="well bg-info text-info" id="function-verbs">**<u>Function:</u> <span class="cmd">verbs</span>**

verbs -- Returns a list of the names of the verbs defined directly on the given <var>object</var>, not inherited from its parent

<div class="well bg-info"><var>list</var> verbs (obj <var>object</var>)</div>

If <var>object</var> is not valid, then <span class="cmd">E_INVARG</span> is raised. If the programmer does not have read permission on <var>object</var>, then <span class="cmd">E_PERM</span> is raised.

Most of the remaining operations on verbs accept a string containing the verb's name to identify the verb in question. Because verbs can have multiple names and because an object can have multiple verbs with the same name, this practice can lead to difficulties. To most unambiguously refer to a particular verb, one can instead use a positive integer, the index of the verb in the list returned by <span class="cmd">verbs()</span>, described above.

For example, suppose that <span class="cmd">verbs(#34)</span> returns this list:

<pre>{"foo", "bar", "baz", "foo"}
</pre>

Object <span class="cmd">#34</span> has two verbs named <span class="cmd">foo</span> defined on it (this may not be an error, if the two verbs have different command syntaxes). To refer unambiguously to the first one in the list, one uses the integer 1; to refer to the other one, one uses 4.

In the function descriptions below, an argument named <var>verb-desc</var> is either a string containing the name of a verb or else a positive integer giving the index of that verb in its defining object's <span class="cmd">verbs()</span> list.

<div class="well bg-warn">For historical reasons, there is also a second, inferior mechanism for referring to verbs with numbers, but its use is strongly discouraged. If the property <span class="cmd">$server_options.support_numeric_verbname_strings</span> exists with a true value, then functions on verbs will also accept a numeric string (e.g., <span class="cmd">"4"</span>) as a verb descriptor. The decimal integer in the string works more-or-less like the positive integers described above, but with two significant differences:

The numeric string is a _zero-based_ index into <span class="cmd">verbs()</span>; that is, in the string case, you would use the number one less than what you would use in the positive integer case.

When there exists a verb whose actual name looks like a decimal integer, this numeric-string notation is ambiguous; the server will in all cases assume that the reference is to the first verb in the list for which the given string could be a name, either in the normal sense or as a numeric index.

Clearly, this older mechanism is more difficult and risky to use; new code should only be written to use the current mechanism, and old code using numeric strings should be modified not to do so.

</div>

</article>

<article class="well bg-info text-info" id="function-verb-info">**<u>Function:</u> <span class="cmd">verb_info</span>**

verb_info -- Get the owner, permission bits, and name(s) for the verb as specified by <var>verb-desc</var> on the given <var>object</var>

<div class="well bg-info"><var>list</var> <span class="cmd">verb_info</span> (obj <var>object</var>, str <var>verb-desc</var>)</div>

**<u>Function:</u> <span class="cmd">set_verb_info</span>**

set_verb_info -- Set the owner, permissions bits, and names(s) for the verb as <var>verb-desc</var> on the given <var>object</var>

<div class="well bg-info"><var>none</var> <span class="cmd">set_verb_info</span> (obj <var>object</var>, str <var>verb-desc</var>, list <var>info</var>)</div>

If <var>object</var> is not valid, then <span class="cmd">E_INVARG</span> is raised. If <var>object</var> does not define a verb as specified by <var>verb-desc</var>, then <span class="cmd">E_VERBNF</span> is raised. If the programmer does not have read (write) permission on the verb in question, then <span class="cmd">verb_info()</span> (<span class="cmd">set_verb_info()</span>) raises <span class="cmd">E_PERM</span>.

Verb info has the following form:

<pre>{<var>owner</var>, <var>perms</var>, <var>names</var>}
</pre>

where <var>owner</var> is an object, <var>perms</var> is a string containing only characters from the set <span class="cmd">r</span>, <span class="cmd">w</span>, <span class="cmd">x</span>, and <span class="cmd">d</span>, and <var>names</var> is a string. This is the kind of value returned by <span class="cmd">verb_info()</span> and expected as the third argument to <span class="cmd">set_verb_info()</span>. <span class="cmd">set_verb_info()</span> raises <span class="cmd">E_INVARG</span> if <var>owner</var> is not valid, if <var>perms</var> contains any illegal characters, or if <var>names</var> is the empty string or consists entirely of spaces; it raises <span class="cmd">E_PERM</span> if <var>owner</var> is not the programmer and the programmer is not a wizard.

</article>

<article class="well bg-info text-info" id="function-verb-args">**<u>Function:</u> <span class="cmd">verb_args</span>**

verb_args -- get the direct-object, preposition, and indirect-object specifications for the verb as specified by <var>verb-desc</var> on the given <var>object</var>.

<div class="well bg-info"><var>list</var> <span class="cmd">verb_args</span> (obj <var>object</var>, str <var>verb-desc</var>)</div>

**<u>Function:</u> <span class="cmd">set_verb_args</span>**

verb_args -- set the direct-object, preposition, and indirect-object specifications for the verb as specified by <var>verb-desc</var> on the given <var>object</var>.

<div class="well bg-info"><var>none</var> <span class="cmd">set_verb_args</span> (obj <var>object</var>, str <var>verb-desc</var>, list <var>args</var>)</div>

If <var>object</var> is not valid, then <span class="cmd">E_INVARG</span> is raised. If <var>object</var> does not define a verb as specified by <var>verb-desc</var>, then <span class="cmd">E_VERBNF</span> is raised. If the programmer does not have read (write) permission on the verb in question, then the function raises <span class="cmd">E_PERM</span>.

Verb args specifications have the following form:

<pre>{<var>dobj</var>, <var>prep</var>, <var>iobj</var>}
</pre>

where <var>dobj</var> and <var>iobj</var> are strings drawn from the set <span class="cmd">"this"</span>, <span class="cmd">"none"</span>, and <span class="cmd">"any"</span>, and <var>prep</var> is a string that is either <span class="cmd">"none"</span>, <span class="cmd">"any"</span>, or one of the prepositional phrases listed much earlier in the description of verbs in the first chapter. This is the kind of value returned by <span class="cmd">verb_args()</span> and expected as the third argument to <span class="cmd">set_verb_args()</span>. Note that for <span class="cmd">set_verb_args()</span>, <var>prep</var> must be only one of the prepositional phrases, not (as is shown in that table) a set of such phrases separated by <span class="cmd">/</span> characters. <span class="cmd">set_verb_args</span> raises <span class="cmd">E_INVARG</span> if any of the <var>dobj</var>, <var>prep</var>, or <var>iobj</var> strings is illegal.

<pre>verb_args($container, "take")
                    =>   {"any", "out of/from inside/from", "this"}
set_verb_args($container, "take", {"any", "from", "this"})
</pre>

</article>

<article class="well bg-info text-info" id="function-add-verb">**<u>Function:</u> <span class="cmd">add_verb</span>**

add_verb -- defines a new verb on the given <var>object</var>

<div class="well bg-info"><var>none</var> <span class="cmd">add_verb</span> (obj <var>object</var>, list <var>info</var>, list <var>args</var>)</div>

The new verb's owner, permission bits and name(s) are given by <var>info</var> in the same format as is returned by <span class="cmd">verb_info()</span>, described above. The new verb's direct-object, preposition, and indirect-object specifications are given by <var>args</var> in the same format as is returned by <span class="cmd">verb_args</span>, described above. The new verb initially has the empty program associated with it; this program does nothing but return an unspecified value.

If <var>object</var> is not valid, or <var>info</var> does not specify a valid owner and well-formed permission bits and verb names, or <var>args</var> is not a legitimate syntax specification, then <span class="cmd">E_INVARG</span> is raised. If the programmer does not have write permission on <var>object</var> or if the owner specified by <var>info</var> is not the programmer and the programmer is not a wizard, then <span class="cmd">E_PERM</span> is raised.

</article>

<article class="well bg-info text-info" id="function-delete-verb">**<u>Function:</u> <span class="cmd">delete_verb</span>**

delete_verb -- removes the verb as specified by <var>verb-desc</var> from the given <var>object</var>

<div class="well bg-info"><var>none</var> <span class="cmd">delete_verb</span> (obj <var>object</var>, str <var>verb-desc</var>)</div>

If <var>object</var> is not valid, then <span class="cmd">E_INVARG</span> is raised. If the programmer does not have write permission on <var>object</var>, then <span class="cmd">E_PERM</span> is raised. If <var>object</var> does not define a verb as specified by <var>verb-desc</var>, then <span class="cmd">E_VERBNF</span> is raised.

</article>

<article class="well bg-info text-info" id="function-verb-code">**<u>Function:</u> <span class="cmd">verb_code</span>**

verb_code -- get the MOO-code program associated with the verb as specified by <var>verb-desc</var> on <var>object</var>

<div class="well bg-info"><var>list</var> <span class="cmd">verb_code</span> (obj <var>object</var>, str <var>verb-desc</var> [, <var>fully-paren</var> [, <var>indent</var>]])</div>

**<u>Function:</u> <span class="cmd">set_verb_code</span>**

set_verb_code -- set the MOO-code program associated with the verb as specified by <var>verb-desc</var> on <var>object</var>

<div class="well bg-info"><var>list</var> <span class="cmd">set_verb_code</span> (obj <var>object</var>, str <var>verb-desc</var>, list <var>code</var>)</div>

The program is represented as a list of strings, one for each line of the program; this is the kind of value returned by <span class="cmd">verb_code()</span> and expected as the third argument to <span class="cmd">set_verb_code()</span>. For <span class="cmd">verb_code()</span>, the expressions in the returned code are usually written with the minimum-necessary parenthesization; if <var>full-paren</var> is true, then all expressions are fully parenthesized.

Also for <span class="cmd">verb_code()</span>, the lines in the returned code are usually not indented at all; if <var>indent</var> is true, each line is indented to better show the nesting of statements.

If <var>object</var> is not valid, then <span class="cmd">E_INVARG</span> is raised. If <var>object</var> does not define a verb as specified by <var>verb-desc</var>, then <span class="cmd">E_VERBNF</span> is raised. If the programmer does not have read (write) permission on the verb in question, then <span class="cmd">verb_code()</span> (<span class="cmd">set_verb_code()</span>) raises <span class="cmd">E_PERM</span>. If the programmer is not, in fact. a programmer, then <span class="cmd">E_PERM</span> is raised.

For <span class="cmd">set_verb_code()</span>, the result is a list of strings, the error messages generated by the MOO-code compiler during processing of <var>code</var>. If the list is non-empty, then <span class="cmd">set_verb_code()</span> did not install <var>code</var>; the program associated with the verb in question is unchanged.

</article>

<article class="well bg-info text-info" id="function-is-disassemble">**<u>Function:</u> <span class="cmd">disassemble</span>**

disassemble -- returns a (longish) list of strings giving a listing of the server's internal "compiled" form of the verb as specified by <var>verb-desc</var> on <var>object</var>

<div class="well bg-info"><var>list</var> <span class="cmd">disassemble</span> (obj <var>object</var>, str <var>verb-desc</var>)</div>

This format is not documented and may indeed change from release to release, but some programmers may nonetheless find the output of <span class="cmd">disassemble()</span> interesting to peruse as a way to gain a deeper appreciation of how the server works.

If <var>object</var> is not valid, then <span class="cmd">E_INVARG</span> is raised. If <var>object</var> does not define a verb as specified by <var>verb-desc</var>, then <span class="cmd">E_VERBNF</span> is raised. If the programmer does not have read permission on the verb in question, then <span class="cmd">disassemble()</span> raises <span class="cmd">E_PERM</span>.

</article>

##### Operations on Player Objects

<article class="well bg-info text-info" id="function-players">**<u>Function:</u> <span class="cmd">players</span>**

players -- returns a list of the object numbers of all player objects in the database

<div class="well bg-info"><var>list</var> <span class="cmd">players</span> ()</div>

</article>

<article class="well bg-info text-info" id="function-is-player">**<u>Function:</u> <span class="cmd">is_player</span>**

is_player -- returns a true value if the given <var>object</var> is a player object and a false value otherwise.

<div class="well bg-info"><var>int</var> <span class="cmd">is_player</span> (obj <var>object</var>)</div>

If <var>object</var> is not valid, <span class="cmd">E_INVARG</span> is raised.

</article>

<article class="well bg-info text-info" id="function-set-player-flag">**<u>Function:</u> <span class="cmd">set_player_flag</span>**

set_player_flag -- confers or removes the "player object" status of the given <var>object</var>, depending upon the truth value of <var>value</var>

<div class="well bg-info"><var>none</var> <span class="cmd">set_player_flag</span> (obj <var>object</var>, <var>value</var>)</div>

If <var>object</var> is not valid, <span class="cmd">E_INVARG</span> is raised. If the programmer is not a wizard, then <span class="cmd">E_PERM</span> is raised.

If <var>value</var> is true, then <var>object</var> gains (or keeps) "player object" status: it will be an element of the list returned by <span class="cmd">players()</span>, the expression <span class="cmd">is_player(<var>object</var>)</span> will return true, and the server will treat a call to <span class="cmd">$do_login_command()</span> that returns <var>object</var> as logging in the current connection.

If <var>value</var> is false, the <var>object</var> loses (or continues to lack) "player object" status: it will not be an element of the list returned by <span class="cmd">players()</span>, the expression <span class="cmd">is_player(<var>object</var>)</span> will return false, and users cannot connect to <var>object</var> by name when they log into the server. In addition, if a user is connected to <var>object</var> at the time that it loses "player object" status, then that connection is immediately broken, just as if <span class="cmd">boot_player(<var>object</var>)</span> had been called (see the description of <span class="cmd">boot_player()</span> below).

</article>

##### Operations on Network Connections

<article class="well bg-info text-info" id="function-connected-players">**<u>Function:</u> <span class="cmd">connected_players</span>**

connected_players -- returns a list of the object numbers of those player objects with currently-active connections

<div class="well bg-info"><var>list</var> <span class="cmd">connected_players</span> ([<var>include-all</var>])</div>

If <var>include-all</var> is provided and true, then the list includes the object numbers associated with _all_ current connections, including ones that are outbound and/or not yet logged-in.

</article>

<article class="well bg-info text-info" id="function-connected-seconds">**<u>Function:</u> <span class="cmd">connected_seconds</span>**

connected_seconds -- return the number of seconds that the currently-active connection to <var>player</var> has existed

<div class="well bg-info"><var>int</var> <span class="cmd">connected_seconds</span> (obj <var>player</var>)</div>

**<u>Function:</u> <span class="cmd">idle_seconds</span>**

idle_seconds -- return the number of seconds that the currently-active connection to <var>player</var> has been idle

<div class="well bg-info"><var>int</var> <span class="cmd">idle_seconds</span> (obj <var>player</var>)</div>

If <var>player</var> is not the object number of a player object with a currently-active connection, then <span class="cmd">E_INVARG</span> is raised.

</article>

<article class="well bg-info text-info" id="function-notify">**<u>Function:</u> <span class="cmd">notify</span>**

notify -- enqueues <var>string</var> for output (on a line by itself) on the connection <var>conn</var>

<div class="well bg-info"><var>none</var> <span class="cmd">notify</span> (obj <var>conn</var>, str <var>string</var> [, <var>no-flush</var>])</div>

If the programmer is not <var>conn</var> or a wizard, then <span class="cmd">E_PERM</span> is raised. If <var>conn</var> is not a currently-active connection, then this function does nothing. Output is normally written to connections only between tasks, not during execution.

The server will not queue an arbitrary amount of output for a connection; the <span class="cmd">MAX_QUEUED_OUTPUT</span> compilation option (in <span class="cmd">options.h</span>) controls the limit. When an attempt is made to enqueue output that would take the server over its limit, it first tries to write as much output as possible to the connection without having to wait for the other end. If that doesn't result in the new output being able to fit in the queue, the server starts throwing away the oldest lines in the queue until the new ouput will fit. The server remembers how many lines of output it has `flushed' in this way and, when next it can succeed in writing anything to the connection, it first writes a line like <span class="cmd">>> Network buffer overflow: <var>X</var> lines of output to you have been lost <<</span> where <var>X</var> is the number of flushed lines.

If <var>no-flush</var> is provided and true, then <span class="cmd">notify()</span> never flushes any output from the queue; instead it immediately returns false. <span class="cmd">Notify()</span> otherwise always returns true.

</article>

<article class="well bg-info text-info" id="function-buffered-output-length">**<u>Function:</u> <span class="cmd">buffered_output_length</span>**

buffered_output_length -- returns the number of bytes currently buffered for output to the connection <var>conn</var>

<div class="well bg-info"><var>int</var> <span class="cmd">buffered_output_length</span> ([obj <var>conn</var>])</div>

If <var>conn</var> is not provided, returns the maximum number of bytes that will be buffered up for output on any connection.

</article>

<article class="well bg-info text-info" id="function-read">**<u>Function:</u> <span class="cmd">read</span>**

read -- reads and returns a line of input from the connection <var>conn</var> (or, if not provided, from the player that typed the command that initiated the current task)

<div class="well bg-info"><var>str</var> <span class="cmd">read</span> ([obj <var>conn</var> [, <var>non-blocking</var>]])</div>

If <var>non-blocking</var> is false or not provided, this function suspends the current task, resuming it when there is input available to be read. If <var>non-blocking</var> is provided and true, this function never suspends the calling task; if there is no input currently available for input, <span class="cmd">read()</span> simply returns 0 immediately.

If <var>player</var> is provided, then the programmer must either be a wizard or the owner of <span class="cmd">player</span>; if <span class="cmd">player</span> is not provided, then <span class="cmd">read()</span> may only be called by a wizard and only in the task that was last spawned by a command from the connection in question. Otherwise, <span class="cmd">E_PERM</span> is raised.

If the given <span class="cmd">player</span> is not currently connected and has no pending lines of input, or if the connection is closed while a task is waiting for input but before any lines of input are received, then <span class="cmd">read()</span> raises <span class="cmd">E_INVARG</span>.

The restriction on the use of <span class="cmd">read()</span> without any arguments preserves the following simple invariant: if input is being read from a player, it is for the task started by the last command that player typed. This invariant adds responsibility to the programmer, however. If your program calls another verb before doing a <span class="cmd">read()</span>, then either that verb must not suspend or else you must arrange that no commands will be read from the connection in the meantime. The most straightforward way to do this is to call

<pre>set_connection_option(player, "hold-input", 1)
</pre>

before any task suspension could happen, then make all of your calls to <span class="cmd">read()</span> and other code that might suspend, and finally call

<pre>set_connection_option(player, "hold-input", 0)
</pre>

to allow commands once again to be read and interpreted normally.

</article>

<article class="well bg-info text-info" id="function-force-input">**<u>Function:</u> <span class="cmd">force_input</span>**

force_input -- inserts the string <var>line</var> as an input task in the queue for the connection <var>conn</var>, just as if it had arrived as input over the network

<div class="well bg-info"><var>none</var> <span class="cmd">force_input</span> (obj <var>conn</var>, str <var>line</var> [, <var>at-front</var>])</div>

If <var>at_front</var> is provided and true, then the new line of input is put at the front of <var>conn</var>'s queue, so that it will be the very next line of input processed even if there is already some other input in that queue. Raises <span class="cmd">E_INVARG</span> if <var>conn</var> does not specify a current connection and <span class="cmd">E_PERM</span> if the programmer is neither <var>conn</var> nor a wizard.

</article>

<article class="well bg-info text-info" id="function-flush-input">**<u>Function:</u> <span class="cmd">flush_input</span>**

flush_input -- performs the same actions as if the connection <var>conn</var>'s defined flush command had been received on that connection

<div class="well bg-info"><var>none</var> <span class="cmd">flush_input</span> (obj <var>conn</var> [<var>show-messages</var>])</div>

I.E., removes all pending lines of input from <var>conn</var>'s queue and, if <var>show-messages</var> is provided and true, prints a message to <var>conn</var> listing the flushed lines, if any. See the chapter on server assumptions about the database for more information about a connection's defined flush command.

</article>

<article class="well bg-info text-info" id="function-output-delimiters">**<u>Function:</u> <span class="cmd">output_delimiters</span>**

output_delimiters -- returns a list of two strings, the current _output prefix_ and _output suffix_ for <var>player</var>.

<div class="well bg-info"><var>list</var> <span class="cmd">output_delimiters</span> (obj <var>player</var>)</div>

If <var>player</var> does not have an active network connection, then <span class="cmd">E_INVARG</span> is raised. If either string is currently undefined, the value <span class="cmd">""</span> is used instead. See the discussion of the <span class="cmd">PREFIX</span> and <span class="cmd">SUFFIX</span> commands in the next chapter for more information about the output prefix and suffix.

</article>

<article class="well bg-info text-info" id="function-boot-player">**<u>Function:</u> <span class="cmd">boot_player</span>**

boot_player -- marks for disconnection any currently-active connection to the given <var>player</var>

<div class="well bg-info"><var>none</var> <span class="cmd">boot_player</span> (obj <var>player</var>)</div>

The connection will not actually be closed until the currently-running task returns or suspends, but all MOO functions (such as <span class="cmd">notify()</span>, <span class="cmd">connected_players()</span>, and the like) immediately behave as if the connection no longer exists. If the programmer is not either a wizard or the same as <var>player</var>, then <span class="cmd">E_PERM</span> is raised. If there is no currently-active connection to <var>player</var>, then this function does nothing.

If there was a currently-active connection, then the following verb call is made when the connection is actually closed:

<pre>$user_disconnected(<var>player</var>)
</pre>

It is not an error if this verb does not exist; the call is simply skipped.

</article>

<article class="well bg-info text-info" id="function-connection-name">**<u>Function:</u> <span class="cmd">connection_name</span>**

connection_name -- returns a network-specific string identifying the connection being used by the given player

<div class="well bg-info"><var>str</var> <span class="cmd">connection_name</span> (obj <var>player</var>)</div>

If the programmer is not a wizard and not <var>player</var>, then <span class="cmd">E_PERM</span> is raised. If <var>player</var> is not currently connected, then <span class="cmd">E_INVARG</span> is raised.

For the TCP/IP networking configurations, for in-bound connections, the string has the form:

<pre>"port <var>lport</var> from <var>host</var>, port <var>port</var>"
</pre>

where <var>lport</var> is the decimal TCP listening port on which the connection arrived, <var>host</var> is either the name or decimal TCP address of the host from which the player is connected, and <var>port</var> is the decimal TCP port of the connection on that host.

For outbound TCP/IP connections, the string has the form

<pre>"port <var>lport</var> to <var>host</var>, port <var>port</var>"
</pre>

where <var>lport</var> is the decimal local TCP port number from which the connection originated, <var>host</var> is either the name or decimal TCP address of the host to which the connection was opened, and <var>port</var> is the decimal TCP port of the connection on that host.

For the System V `local' networking configuration, the string is the UNIX login name of the connecting user or, if no such name can be found, something of the form:

<pre>"User #<var>number</var>"
</pre>

where <var>number</var> is a UNIX numeric user ID.

For the other networking configurations, the string is the same for all connections and, thus, useless.

</article>

<article class="well bg-info text-info" id="function-set-connection-option">**<u>Function:</u> <span class="cmd">set_connection_option</span>**

set_connection_option -- controls a number of optional behaviors associated the connection <var>conn</var>

<div class="well bg-info"><var>none</var> <span class="cmd">set_connection_option</span> (obj <var>conn</var>, str <var>option</var>, <var>value</var>)</div>

Raises <span class="cmd">E_INVARG</span> if <var>conn</var> does not specify a current connection and <span class="cmd">E_PERM</span> if the programmer is neither <var>conn</var> nor a wizard.

The following values for <var>option</var> are currently supported:

<span class="cmd">"hold-input"</span>  

If <var>value</var> is true, then input received on <var>conn</var> will never be treated as a command; instead, it will remain in the queue until retrieved by a call to <span class="cmd">read()</span>.

<span class="cmd">"client-echo"</span>  
Send the Telnet Protocol <span class="cmd">WONT ECHO</span> or <span class="cmd">WILL ECHO</span> command, depending on whether <var>value</var> is true or false, respectively. For clients that support the Telnet Protocol, this should toggle whether or not the client echoes locally the characters typed by the user. Note that the server itself never echoes input characters under any circumstances. (This option is only available under the TCP/IP networking configurations.)

<span class="cmd">"binary"</span>  
If <var>value</var> is true, then both input from and output to <var>conn</var> can contain arbitrary bytes. Input from a connection in binary mode is not broken into lines at all; it is delivered to either the read() function or the built-in command parser as _binary strings_, in whatever size chunks come back from the operating system. (See the early section on MOO value types for a description of the binary string representation.) For output to a connection in binary mode, the second argument to `notify()' must be a binary string; if it is malformed, E_INVARG is raised.

<span class="cmd">"flush-command"</span>  
If <var>value</var> is a non-empty string, then it becomes the new _flush_ command for this connection, by which the player can flush all queued input that has not yet been processed by the server. If <var>value</var> is not a non-empty string, then <var>conn</var> is set to have no flush command at all. The default value of this option can be set via the property <span class="cmd">$server_options.default_flush_command</span>; see the chapter on server assumptions about the database for details.

</article>

<article class="well bg-info text-info" id="function-connection-options">**<u>Function:</u> <span class="cmd">connection_options</span>**

connection_options -- returns a list of <span class="cmd">{<var>name</var>, <var>value</var>}</span> pairs describing the current settings of all of the allowed options for the connection <var>conn</var>

<div class="well bg-info"><var>list</var> <span class="cmd">connection_options</span> (obj <var>conn</var>)</div>

Raises <span class="cmd">E_INVARG</span> if <var>conn</var> does not specify a current connection and <span class="cmd">E_PERM</span> if the programmer is neither <var>conn</var> nor a wizard.

</article>

<article class="well bg-info text-info" id="function-connection_option">**<u>Function:</u> <span class="cmd">connection_option</span>**

connection_option -- returns the current setting of the option <var>name</var> for the connection <var>conn</var>

<div class="well bg-info"><var>value</var> <span class="cmd">>connection_option</span> (obj <var>conn</var>, str <var>name</var>)</div>

Raises <span class="cmd">E_INVARG</span> if <var>conn</var> does not specify a current connection and <span class="cmd">E_PERM</span> if the programmer is neither <var>conn</var> nor a wizard.

</article>

<article class="well bg-info text-info" id="function-open-network-connection">**<u>Function:</u> <span class="cmd">open_network_connection</span>**

open_network_connection -- establishes a network connection to the place specified by the arguments and more-or-less pretends that a new, normal player connection has been established from there

<div class="well bg-info"><var>obj</var> <span class="cmd">open_network_connection</span> (<var>value</var>, ...)</div>

The new connection, as usual, will not be logged in initially and will have a negative object number associated with it for use with <span class="cmd">read()</span>, <span class="cmd">notify()</span>, and <span class="cmd">boot_player()</span>. This object number is the value returned by this function.

If the programmer is not a wizard or if the <span class="cmd">OUTBOUND_NETWORK</span> compilation option was not used in building the server, then <span class="cmd">E_PERM</span> is raised. If the network connection cannot be made for some reason, then other errors will be returned, depending upon the particular network implementation in use.

For the TCP/IP network implementations (the only ones as of this writing that support outbound connections), there must be two arguments, a string naming a host (possibly using the numeric Internet syntax) and an integer specifying a TCP port. If a connection cannot be made because the host does not exist, the port does not exist, the host is not reachable or refused the connection, <span class="cmd">E_INVARG</span> is raised. If the connection cannot be made for other reasons, including resource limitations, then <span class="cmd">E_QUOTA</span> is raised.

The outbound connection process involves certain steps that can take quite a long time, during which the server is not doing anything else, including responding to user commands and executing MOO tasks. See the chapter on server assumptions about the database for details about how the server limits the amount of time it will wait for these steps to successfully complete.

It is worth mentioning one tricky point concerning the use of this function. Since the server treats the new connection pretty much like any normal player connection, it will naturally try to parse any input from that connection as commands in the usual way. To prevent this treatment, you should use <span class="cmd">set_connection_option()</span> to set the <span class="cmd">"hold-input"</span> option true on the connection.

</article>

<article class="well bg-info text-info" id="function-listen">**<u>Function:</u> <span class="cmd">listen</span>**

listen -- create a new point at which the server will listen for network connections, just as it does normally

<div class="well bg-info"><var>value</var> <span class="cmd">listen</span> (obj <var>object</var>, <var>point</var> [, <var>print-messages</var>])</div>

<var>Object</var> is the object whose verbs <span class="cmd">do_login_command</span>, <span class="cmd">do_command</span>, <span class="cmd">do_out_of_band_command</span>, <span class="cmd">user_connected</span>, <span class="cmd">user_created</span>, <span class="cmd">user_reconnected</span>, <span class="cmd">user_disconnected</span>, and <span class="cmd">user_client_disconnected</span> will be called at appropriate points, just as these verbs are called on <span class="cmd">#0</span> for normal connections. (See the chapter on server assumptions about the database for the complete story on when these functions are called.) <var>Point</var> is a network-configuration-specific parameter describing the listening point. If <var>print-messages</var> is provided and true, then the various database-configurable messages (also detailed in the chapter on server assumptions) will be printed on connections received at the new listening point. <span class="cmd">Listen()</span> returns <var>canon</var>, a `canonicalized' version of <var>point</var>, with any configuration-specific defaulting or aliasing accounted for.

This raises <span class="cmd">E_PERM</span> if the programmer is not a wizard, <span class="cmd">E_INVARG</span> if <var>object</var> is invalid or there is already a listening point described by <var>point</var>, and <span class="cmd">E_QUOTA</span> if some network-configuration-specific error occurred.

For the TCP/IP configurations, <var>point</var> is a TCP port number on which to listen and <var>canon</var> is equal to <var>point</var> unless <var>point</var> is zero, in which case <var>canon</var> is a port number assigned by the operating system.

For the local multi-user configurations, <var>point</var> is the UNIX file name to be used as the connection point and <var>canon</var> is always equal to <var>point</var>.

In the single-user configuration, the can be only one listening point at a time; <var>point</var> can be any value at all and <var>canon</var> is always zero.

</article>

<article class="well bg-info text-info" id="function-unlisten">**<u>Function:</u> <span class="cmd">unlisten</span>**

unlisten -- stop listening for connections on the point described by <var>canon</var>, which should be the second element of some element of the list returned by <span class="cmd">listeners()</span>

<div class="well bg-info"><var>none</var> <span class="cmd">unlisten</span> (<var>canon</var>)</div>

Raises <span class="cmd">E_PERM</span> if the programmer is not a wizard and <span class="cmd">E_INVARG</span> if there does not exist a listener with that description.

</article>

<article class="well bg-info text-info" id="function-listeners">**<u>Function:</u> <span class="cmd">listeners</span>**

listeners -- returns a list describing all existing listening points, including the default one set up automatically by the server when it was started (unless that one has since been destroyed by a call to <span class="cmd">unlisten()</span>)

<div class="well bg-info"><var>list</var> <span class="cmd">listeners</span> ()</div>

Each element of the list has the following form:

<pre>{<var>object</var>, <var>canon</var>, <var>print-messages</var>}
</pre>

where <var>object</var> is the first argument given in the call to <span class="cmd">listen()</span> to create this listening point, <var>print-messages</var> is true if the third argument in that call was provided and true, and <var>canon</var> was the value returned by that call. (For the initial listening point, <var>object</var> is <span class="cmd">#0</span>, <var>canon</var> is determined by the command-line arguments or a network-configuration-specific default, and <var>print-messages</var> is true.)

Please note that there is nothing special about the initial listening point created by the server when it starts; you can use <span class="cmd">unlisten()</span> on it just as if it had been created by <span class="cmd">listen()</span>. This can be useful; for example, under one of the TCP/IP configurations, you might start up your server on some obscure port, say 12345, connect to it by yourself for a while, and then open it up to normal users by evaluating the statments:

<pre>unlisten(12345); listen(#0, 7777, 1)
</pre>

</article>

##### Operations Involving Times and Dates

<article class="well bg-info text-info" id="function-time">**<u>Function:</u> <span class="cmd">time</span>**

time -- returns the current time, represented as the number of seconds that have elapsed since midnight on 1 January 1970, Greenwich Mean Time

<div class="well bg-info"><var>int</var> <span class="cmd">time</span> ()</div>

</article>

<article class="well bg-info text-info" id="function-ctime">**<u>Function:</u> <span class="cmd">ctime</span>**

ctime -- interprets <var>time</var> as a time, using the same representation as given in the description of <span class="cmd">time()</span>, above, and converts it into a 28-character, human-readable string

<div class="well bg-info"><var>str</var> <span class="cmd">ctime</span> ([int <var>time</var>])</div>

The string will be in the following format:

<pre>Mon Aug 13 19:13:20 1990 PDT
</pre>

If the current day of the month is less than 10, then an extra blank appears between the month and the day:

<pre>Mon Apr  1 14:10:43 1991 PST
</pre>

If <var>time</var> is not provided, then the current time is used.

Note that <span class="cmd">ctime()</span> interprets <var>time</var> for the local time zone of the computer on which the MOO server is running.

</article>

##### MOO-Code Evaluation and Task Manipulation

<article class="well bg-info text-info" id="function-raise">**<u>Function:</u> <span class="cmd">raise</span>**

raise -- raises <var>code</var> as an error in the same way as other MOO expressions, statements, and functions do

<div class="well bg-info"><var>none</var> <span class="cmd">raise</span> (<var>code</var> [, str <var>message</var> [, <var>value</var>]])</div>

<var>Message</var>, which defaults to the value of <span class="cmd">tostr(<var>code</var>)</span>, and <var>value</var>, which defaults to zero, are made available to any <span class="cmd">try</span>-<span class="cmd">except</span> statements that catch the error. If the error is not caught, then <var>message</var> will appear on the first line of the traceback printed to the user.

</article>

<article class="well bg-info text-info" id="function-call-function">**<u>Function:</u> <span class="cmd">call_function</span>**

call_function -- calls the built-in function named <var>func-name</var>, passing the given arguments, and returns whatever that function returns

<div class="well bg-info"><var>value</var> <span class="cmd">call_function</span> (str <var>func-name</var>, <var>arg</var>, ...)</div>

Raises <span class="cmd">E_INVARG</span> if <var>func-name</var> is not recognized as the name of a known built-in function. This allows you to compute the name of the function to call and, in particular, allows you to write a call to a built-in function that may or may not exist in the particular version of the server you're using.

</article>

<article class="well bg-info text-info" id="function-function-info">**<u>Function:</u> <span class="cmd">function_info</span>**

function_info -- returns descriptions of the built-in functions available on the server

<div class="well bg-info"><var>list</var> <span class="cmd">function_info</span> ([str <var>name</var>])</div>

If <var>name</var> is provided, only the description of the function with that name is returned. If <var>name</var> is omitted, a list of descriptions is returned, one for each function available on the server. Raised <span class="cmd">E_INVARG</span> if <var>name</var> is provided but no function with that name is available on the server.

Each function description is a list of the following form:

<pre>{<var>name</var>, <var>min-args</var>, <var>max-args</var>, <var>types</var>
</pre>

where <var>name</var> is the name of the built-in function, <var>min-args</var> is the minimum number of arguments that must be provided to the function, <var>max-args</var> is the maximum number of arguments that can be provided to the function or <span class="cmd">-1</span> if there is no maximum, and <var>types</var> is a list of <var>max-args</var> integers (or <var>min-args</var> if <var>max-args</var> is <span class="cmd">-1</span>), each of which represents the type of argument required in the corresponding position. Each type number is as would be returned from the <span class="cmd">typeof()</span> built-in function except that <span class="cmd">-1</span> indicates that any type of value is acceptable and <span class="cmd">-2</span> indicates that either integers or floating-point numbers may be given. For example, here are several entries from the list:

<pre>{"listdelete", 2, 2, {4, 0}}
{"suspend", 0, 1, {0}}
{"server_log", 1, 2, {2, -1}}
{"max", 1, -1, {-2}}
{"tostr", 0, -1, {}}
</pre>

<span class="cmd">listdelete()</span> takes exactly 2 arguments, of which the first must be a list (<span class="cmd">LIST == 4</span>) and the second must be an integer (<span class="cmd">INT == 0</span>). <span class="cmd">Suspend()</span> has one optional argument that, if provided, must be an integer. <span class="cmd">Server_log()</span> has one required argument that must be a string (<span class="cmd">STR == 2</span>) and one optional argument that, if provided, may be of any type. <span class="cmd">max()</span> requires at least one argument but can take any number above that, and the first argument must be either an integer or a floating-point number; the type(s) required for any other arguments can't be determined from this description. Finally, <span class="cmd">tostr()</span> takes any number of arguments at all, but it can't be determined from this description which argument types would be acceptable in which positions.

</article>

<article class="well bg-info text-info" id="function-is-eval">**<u>Function:</u> <span class="cmd">eval</span>**

eval -- the MOO-code compiler processes <var>string</var> as if it were to be the program associated with some verb and, if no errors are found, that fictional verb is invoked

<div class="well bg-info"><var>list</var> <span class="cmd">eval</span> (str <var>string</var>)</div>

If the programmer is not, in fact, a programmer, then <span class="cmd">E_PERM</span> is raised. The normal result of calling <span class="cmd">eval()</span> is a two element list. The first element is true if there were no compilation errors and false otherwise. The second element is either the result returned from the fictional verb (if there were no compilation errors) or a list of the compiler's error messages (otherwise).

When the fictional verb is invoked, the various built-in variables have values as shown below:

<pre>player    the same as in the calling verb
this      #-1
caller    the same as the initial value of <span class="cmd">this</span> in the calling verb

args      {}
argstr    ""

verb      ""
dobjstr   ""
dobj      #-1
prepstr   ""
iobjstr   ""
iobj      #-1
</pre>

The fictional verb runs with the permissions of the programmer and as if its <span class="cmd">d</span> permissions bit were on.

<pre>eval("return 3 + 4;")   =>   {1, 7}
</pre>

</article>

<article class="well bg-info text-info" id="function-set-task-perms">**<u>Function:</u> <span class="cmd">set_task_perms</span>**

set_task_perms -- changes the permissions with which the currently-executing verb is running to be those of <var>who</var>

<div class="well bg-info"><var>one</var> <span class="cmd">set_task_perms</span> (obj <var>who</var>)</div>

If the programmer is neither <var>who</var> nor a wizard, then <span class="cmd">E_PERM</span> is raised.

<div class="well"><span class="label label-info">Note:</span> This does not change the owner of the currently-running verb, only the permissions of this particular invocation. It is used in verbs owned by wizards to make themselves run with lesser (usually non-wizard) permissions.</div>

</article>

<article class="well bg-info text-info" id="function-caller-perms">**<u>Function:</u> <span class="cmd">caller_perms</span>**

caller_perms -- returns the permissions in use by the verb that called the currently-executing verb

<div class="well bg-info"><var>obj</var> <span class="cmd">caller_perms</span> ()</div>

If the currently-executing verb was not called by another verb (i.e., it is the first verb called in a command or server task), then <span class="cmd">caller_perms()</span> returns <span class="cmd">#-1</span>.

</article>

<article class="well bg-info text-info" id="function-ticks-left">**<u>Function:</u> <span class="cmd">ticks_left</span>**

ticks_left -- return the number of ticks left to the current task before it will be forcibly terminated

<div class="well bg-info"><var>int</var> <span class="cmd">ticks_left</span> ()</div>

**<u>Function:</u> <span class="cmd">seconds_left</span>**

seconds_left -- return the number of seconds left to the current task before it will be forcibly terminated

<div class="well bg-info"><var>int</var> <span class="cmd">seconds_left</span> ()</div>

These are useful, for example, in deciding when to call <span class="cmd">suspend()</span> to continue a long-lived computation.

</article>

<article class="well bg-info text-info" id="function-task-id">**<u>Function:</u> <span class="cmd">task_id</span>**

task_id -- returns the non-zero, non-negative integer identifier for the currently-executing task

<div class="well bg-info"><var>int</var> <span class="cmd">task_id</span> ()</div>

Such integers are randomly selected for each task and can therefore safely be used in circumstances where unpredictability is required.

</article>

<article class="well bg-info text-info" id="function-suspend">**<u>Function:</u> <span class="cmd">suspend</span>**

suspend -- suspends the current task, and resumes it after at least <var>seconds</var> seconds

<div class="well bg-info"><var>value</var> <span class="cmd">suspend</span> ([int <var>seconds</var>])</div>

If <var>seconds</var> is not provided, the task is suspended indefinitely; such a task can only be resumed by use of the <span class="cmd">resume()</span> function.

When the task is resumed, it will have a full quota of ticks and seconds. This function is useful for programs that run for a long time or require a lot of ticks. If <var>seconds</var> is negative, then <span class="cmd">E_INVARG</span> is raised. <span class="cmd">Suspend()</span> returns zero unless it was resumed via <span class="cmd">resume()</span>, in which case it returns the second argument given to that function.

In some sense, this function forks the `rest' of the executing task. However, there is a major difference between the use of <span class="cmd">suspend(<var>seconds</var>)</span> and the use of the <span class="cmd">fork (<var>seconds</var>)</span>. The <span class="cmd">fork</span> statement creates a new task (a _forked task_) while the currently-running task still goes on to completion, but a <span class="cmd">suspend()</span> suspends the currently-running task (thus making it into a _suspended task_). This difference may be best explained by the following examples, in which one verb calls another:

<pre>.program   #0:caller_A
#0.prop = 1;
#0:callee_A();
#0.prop = 2;
.

.program   #0:callee_A
fork(5)
  #0.prop = 3;
endfork
.

.program   #0:caller_B
#0.prop = 1;
#0:callee_B();
#0.prop = 2;
.

.program   #0:callee_B
suspend(5);
#0.prop = 3;
.
</pre>

Consider <span class="cmd">#0:caller_A</span>, which calls <span class="cmd">#0:callee_A</span>. Such a task would assign 1 to <span class="cmd">#0.prop</span>, call <span class="cmd">#0:callee_A</span>, fork a new task, return to <span class="cmd">#0:caller_A</span>, and assign 2 to <span class="cmd">#0.prop</span>, ending this task. Five seconds later, if the forked task had not been killed, then it would begin to run; it would assign 3 to <span class="cmd">#0.prop</span> and then stop. So, the final value of <span class="cmd">#0.prop</span> (i.e., the value after more than 5 seconds) would be 3.

Now consider <span class="cmd">#0:caller_B</span>, which calls <span class="cmd">#0:callee_B</span> instead of <span class="cmd">#0:callee_A</span>. This task would assign 1 to <span class="cmd">#0.prop</span>, call <span class="cmd">#0:callee_B</span>, and suspend. Five seconds later, if the suspended task had not been killed, then it would resume; it would assign 3 to <span class="cmd">#0.prop</span>, return to <span class="cmd">#0:caller_B</span>, and assign 2 to <span class="cmd">#0.prop</span>, ending the task. So, the final value of <span class="cmd">#0.prop</span> (i.e., the value after more than 5 seconds) would be 2.

A suspended task, like a forked task, can be described by the <span class="cmd">queued_tasks()</span> function and killed by the <span class="cmd">kill_task()</span> function. Suspending a task does not change its task id. A task can be suspended again and again by successive calls to <span class="cmd">suspend()</span>.

By default, there is no limit to the number of tasks any player may suspend, but such a limit can be imposed from within the database. See the chapter on server assumptions about the database for details.

</article>

<article class="well bg-info text-info" id="function-resume">**<u>Function:</u> <span class="cmd">resume</span>**

resume -- immediately ends the suspension of the suspended task with the given <var>task-id</var>; that task's call to <span class="cmd">suspend()</span> will return <var>value</var>, which defaults to zero

<div class="well bg-info"><var>none</var> <span class="cmd">resume</span> (int <var>task-id</var> [, <var>value</var>])</div>

If <var>value</var> is of type <span class="cmd">ERR</span>, it will be raised, rather than returned, in the suspended task. <span class="cmd">Resume()</span> raises <span class="cmd">E_INVARG</span> if <var>task-id</var> does not specify an existing suspended task and <span class="cmd">E_PERM</span> if the programmer is neither a wizard nor the owner of the specified task.

</article>

<article class="well bg-info text-info" id="function-queue-info">**<u>Function:</u> <span class="cmd">queue_info</span>**

queue_info -- if <var>player</var> is omitted, returns a list of object numbers naming all players that currently have active task queues inside the server

<div class="well bg-info"><var>list</var> <span class="cmd">queue_info</span> ([obj <var>player</var>])</div>

If <var>player</var> is provided, returns the number of background tasks currently queued for that user. It is guaranteed that <span class="cmd">queue_info(<var>X</var>)</span> will return zero for any <var>X</var> not in the result of <span class="cmd">queue_info()</span>.

</article>

<article class="well bg-info text-info" id="function-queued-tasks">**<u>Function:</u> <span class="cmd">queued_tasks</span>**

queued_tasks -- returns information on each of the background tasks (i.e., forked, suspended or reading) owned by the programmer (or, if the programmer is a wizard, all queued tasks)

<div class="well bg-info"><var>list</var> <span class="cmd">queued_tasks</span> ()</div>

The returned value is a list of lists, each of which encodes certain information about a particular queued task in the following format:

<pre>{<var>task-id</var>, <var>start-time</var>, <var>x</var>, <var>y</var>,
 <var>programmer</var>, <var>verb-loc</var>, <var>verb-name</var>, <var>line</var>, <var>this</var>}
</pre>

where <var>task-id</var> is an integer identifier for this queued task, <var>start-time</var> is the time after which this task will begin execution (in <span class="cmd">time()</span> format), <var>x</var> and <var>y</var> are obsolete values that are no longer interesting, <var>programmer</var> is the permissions with which this task will begin execution (and also the player who _owns_ this task), <var>verb-loc</var> is the object on which the verb that forked this task was defined at the time, <var>verb-name</var> is that name of that verb, <var>line</var> is the number of the first line of the code in that verb that this task will execute, and <var>this</var> is the value of the variable <span class="cmd">this</span> in that verb.

For reading tasks, <var>start-time</var> is <span class="cmd">-1</span>.

The <var>x</var> and <var>y</var> fields are now obsolete and are retained only for backward-compatibility reasons. They may be reused for new purposes in some future version of the server.

</article>

<article class="well bg-info text-info" id="function-kill-task">**<u>Function:</u> <span class="cmd">kill_task</span>**

kill_task -- removes the task with the given <var>task-id</var> from the queue of waiting tasks

<div class="well bg-info"><var>none</var> <span class="cmd">kill_task</span> (int <var>task-id</var>)</div>

If the programmer is not the owner of that task and not a wizard, then <span class="cmd">E_PERM</span> is raised. If there is no task on the queue with the given <var>task-id</var>, then <span class="cmd">E_INVARG</span> is raised.

</article>

<article class="well bg-info text-info" id="function-callers">**<u>Function:</u> <span class="cmd">callers</span>**

callers -- returns information on each of the verbs and built-in functions currently waiting to resume execution in the current task

<div class="well bg-info"><var>list</var> <span class="cmd">callers</span> ([<var>include-line-numbers</var>])</div>

When one verb or function calls another verb or function, execution of the caller is temporarily suspended, pending the called verb or function returning a value. At any given time, there could be several such pending verbs and functions: the one that called the currently executing verb, the verb or function that called that one, and so on. The result of <span class="cmd">callers()</span> is a list, each element of which gives information about one pending verb or function in the following format:

<pre>{<var>this</var>, <var>verb-name</var>, <var>programmer</var>, <var>verb-loc</var>, <var>player</var>, <var>line-number</var>}
</pre>

For verbs, <var>this</var> is the initial value of the variable <span class="cmd">this</span> in that verb, <var>verb-name</var> is the name used to invoke that verb, <var>programmer</var> is the player with whose permissions that verb is running, <var>verb-loc</var> is the object on which that verb is defined, <var>player</var> is the initial value of the variable <span class="cmd">player</span> in that verb, and <var>line-number</var> indicates which line of the verb's code is executing. The <var>line-number</var> element is included only if the <var>include-line-numbers</var> argument was provided and true.

For functions, <var>this</var>, <var>programmer</var>, and <var>verb-loc</var> are all <span class="cmd">#-1</span>, <var>verb-name</var> is the name of the function, and <var>line-number</var> is an index used internally to determine the current state of the built-in function. The simplest correct test for a built-in function entry is

<pre>(VERB-LOC == #-1  &&  PROGRAMMER == #-1  &&  VERB-name != "")
</pre>

The first element of the list returned by <span class="cmd">callers()</span> gives information on the verb that called the currently-executing verb, the second element describes the verb that called that one, and so on. The last element of the list describes the first verb called in this task.

</article>

<article class="well bg-info text-info" id="function-task-stack">**<u>Function:</u> <span class="cmd">task_stack</span>**

task_stack -- returns information like that returned by the <span class="cmd">callers()</span> function, but for the suspended task with the given <var>task-id</var>; the <var>include-line-numbers</var> argument has the same meaning as in <span class="cmd">callers()</span>

<div class="well bg-info"><var>list</var> <span class="cmd">task_stack</span> (int <var>task-id</var> [, <var>include-line-numbers</var>])</div>

Raises <span class="cmd">E_INVARG</span> if <var>task-id</var> does not specify an existing suspended task and <span class="cmd">E_PERM</span> if the programmer is neither a wizard nor the owner of the specified task.

</article>

##### Administrative Operations

<article class="well bg-info text-info" id="function-server-version">**<u>Function:</u> <span class="cmd">server_version</span>**

server_version -- returns a string giving the version number of the running MOO server

<div class="well bg-info"><var>str</var> <span class="cmd">server_version</span> ()</div>

</article>

<article class="well bg-info text-info" id="function-server-log">**<u>Function:</u> <span class="cmd">server_log</span>**

server_log -- the text in <var>message</var> is sent to the server log with a distinctive prefix (so that it can be distinguished from server-generated messages)

<div class="well bg-info"><var>none</var> <span style="cmd">server_log</span> (str <var>message</var> [, <var>is-error</var>])</div>

If the programmer is not a wizard, then <span class="cmd">E_PERM</span> is raised. If <var>is-error</var> is provided and true, then <var>message</var> is marked in the server log as an error.

</article>

<article class="well bg-info text-info" id="function-renumber">**<u>Function:</u> <span class="cmd">renumber</span>**

renumber -- the object number of the object currently numbered <var>object</var> is changed to be the least nonnegative object number not currently in use and the new object number is returned

<div class="well bg-info"><var>obj</var> <span class="cmd">renumber</span> (obj <var>object</var>)</div>

If <var>object</var> is not valid, then <span class="cmd">E_INVARG</span> is raised. If the programmer is not a wizard, then <span class="cmd">E_PERM</span> is raised. If there are no unused nonnegative object numbers less than <var>object</var>, then <var>object</var> is returned and no changes take place.

The references to <var>object</var> in the parent/children and location/contents hierarchies are updated to use the new object number, and any verbs, properties and/or objects owned by <var>object</var> are also changed to be owned by the new object number. The latter operation can be quite time consuming if the database is large. No other changes to the database are performed; in particular, no object references in property values or verb code are updated.

This operation is intended for use in making new versions of the LambdaCore database from the then-current LambdaMOO database, and other similar situations. Its use requires great care.

</article>

<article class="well bg-info text-info" id="function-reset-max-object">**<u>Function:</u> <span class="cmd">reset_max_object</span>**

reset_max_object -- the server's idea of the highest object number ever used is changed to be the highest object number of a currently-existing object, thus allowing reuse of any higher numbers that refer to now-recycled objects

<div class="well bg-info"><var>none</var> <span class="cmd">reset_max_object</span> ()</div>

If the programmer is not a wizard, then <span class="cmd">E_PERM</span> is raised.

This operation is intended for use in making new versions of the LambdaCore database from the then-current LambdaMOO database, and other similar situations. Its use requires great care.

</article>

<article class="well bg-info text-info" id="function-memory-usage">**<u>Function:</u> <span class="cmd">memory_usage</span>**

memory_usage -- on some versions of the server, this returns statistics concerning the server consumption of system memory

<div class="well bg-info"><var>list</var> <span class="cmd">memory_usage</span> ()</div>

The result is a list of lists, each in the following format:

<pre>{<var>block-size</var>, <var>nused</var>, <var>nfree</var>}
</pre>

where <var>block-size</var> is the size in bytes of a particular class of memory fragments, <var>nused</var> is the number of such fragments currently in use in the server, and <var>nfree</var> is the number of such fragments that have been reserved for use but are currently free.

On servers for which such statistics are not available, <span class="cmd">memory_usage()</span> returns <span class="cmd">{}</span>. The compilation option <span class="cmd">USE_GNU_MALLOC</span> controls whether or not statistics are available; if the option is not provided, statistics are not available.

</article>

<article class="well bg-info text-info" id="function-dump-database">**<u>Function:</u> <span class="cmd">dump_database</span>**

dump_database -- requests that the server checkpoint the database at its next opportunity

<div class="well bg-info"><var>none</var> <span class="cmd">dump_database</span> ()</div>

It is not normally necessary to call this function; the server automatically checkpoints the database at regular intervals; see the chapter on server assumptions about the database for details. If the programmer is not a wizard, then <span class="cmd">E_PERM</span> is raised.

</article>

<article class="well bg-info text-info" id="function-db-disk-size">**<u>Function:</u> <span class="cmd">db_disk_size</span>**

db_disk_size -- returns the total size, in bytes, of the most recent full representation of the database as one or more disk files

<div class="well bg-info"><var>int</var> <span class="cmd">db_disk_size</span> ()</div>

Raises <span class="cmd">E_QUOTA</span> if, for some reason, no such on-disk representation is currently available.

</article>

<article class="well bg-info text-info" id="function-shutdown">**<u>Function:</u> <span class="cmd">shutdown</span>**

shutdown -- requests that the server shut itself down at its next opportunity

<div class="well bg-info"><var>none</var> <span class="cmd">shutdown</span> ([str <var>message</var>])</div>

Before doing so, a notice (incorporating <var>message</var>, if provided) is printed to all connected players. If the programmer is not a wizard, then <span class="cmd">E_PERM</span> is raised.

</article>

#### Server Commands and Database Assumptions

This chapter describes all of the commands that are built into the server and every property and verb in the database specifically accessed by the server. Aside from what is listed here, no assumptions are made by the server concerning the contents of the database.

## [Built-in Commands](ProgrammersManual_toc.html#TOC57)

As was mentioned in the chapter on command parsing, there are five commands whose interpretation is fixed by the server: <span class="cmd">PREFIX</span>, <span class="cmd">OUTPUTPREFIX</span>, <span class="cmd">SUFFIX</span>, <span class="cmd">OUTPUTSUFFIX</span>, and <span class="cmd">.program</span>. The first four of these are intended for use by programs that connect to the MOO, so-called `client' programs. The <span class="cmd">.program</span> command is used by programmers to associate a MOO program with a particular verb. The server can, in addition, recognize a sixth special command on any or all connections, the _flush_ command.

The server also performs special processing on command lines that begin with certain punctuation characters.

This section discusses these built-in pieces of the command-interpretation process.

### [Command-Output Delimiters](ProgrammersManual_toc.html#TOC58)

Every MOO network connection has associated with it two strings, the _output prefix_ and the _output suffix_. Just before executing a command typed on that connection, the server prints the output prefix, if any, to the player. Similarly, just after finishing the command, the output suffix, if any, is printed to the player. Initially, these strings are not defined, so no extra printing takes place.

The <span class="cmd">PREFIX</span> and <span class="cmd">SUFFIX</span> commands are used to set and clear these strings. They have the following simple syntax:

<pre>PREFIX  <var>output-prefix</var>
SUFFIX  <var>output-suffix</var>
</pre>

That is, all text after the command name and any following spaces is used as the new value of the appropriate string. If there is no non-blank text after the command string, then the corresponding string is cleared. For compatibility with some general MUD client programs, the server also recognizes <span class="cmd">OUTPUTPREFIX</span> as a synonym for <span class="cmd">PREFIX</span> and <span class="cmd">OUTPUTSUFFIX</span> as a synonym for <span class="cmd">SUFFIX</span>.

These commands are intended for use by programs connected to the MOO, so that they can issue MOO commands and reliably determine the beginning and end of the resulting output. For example, one editor-based client program sends this sequence of commands on occasion:

<pre>PREFIX >>MOO-Prefix<<
SUFFIX >>MOO-Suffix<<
@list <var>object</var>:<var>verb</var> without numbers
PREFIX
SUFFIX
</pre>

The effect of which, in a LambdaCore-derived database, is to print out the code for the named verb preceded by a line containing only <span class="cmd">>>MOO-Prefix<<</span> and followed by a line containing only <span class="cmd">>>MOO-Suffix<<</span>. This enables the editor to reliably extract the program text from the MOO output and show it to the user in a separate editor window. There are many other possible uses.

The built-in function <span class="cmd">output_delimiters()</span> can be used by MOO code to find out the output prefix and suffix currently in effect on a particular network connection.

### [Programming](ProgrammersManual_toc.html#TOC59)

The <span class="cmd">.program</span> command is a common way for programmers to associate a particular MOO-code program with a particular verb. It has the following syntax:

<pre>.program <var>object</var>:<var>verb</var>
...<var>several lines of MOO code</var>...
.
</pre>

That is, after typing the <span class="cmd">.program</span> command, then all lines of input from the player are considered to be a part of the MOO program being defined. This ends as soon as the player types a line containing only a dot (<span class="cmd">.</span>). When that line is received, the accumulated MOO program is checked for proper MOO syntax and, if correct, associated with the named verb.

If, at the time the line containing only a dot is processed, (a) the player is not a programmer, (b) the player does not have write permission on the named verb, or (c) the property <span class="cmd">$server_options.protect_set_verb_code</span> exists and has a true value and the player is not a wizard, then an error message is printed and the named verb's program is not changed.

In the <span class="cmd">.program</span> command, <var>object</var> may have one of three forms:

*   The name of some object visible to the player. This is exactly like the kind of matching done by the server for the direct and indirect objects of ordinary commands. See the chapter on command parsing for details. Note that the special names <span class="cmd">me</span> and <span class="cmd">here</span> may be used.
*   An object number, in the form <span class="cmd">#<var>number</var></span>.
*   A _system property_ (that is, a property on <span class="cmd">#0</span>), in the form <span class="cmd">$<var>name</var></span>. In this case, the current value of <span class="cmd">#0.<var>name</var></span> must be a valid object.

### [Flushing Unprocessed Input](ProgrammersManual_toc.html#TOC60)

It sometimes happens that a user changes their mind about having typed one or more lines of input and would like to `untype' them before the server actually gets around to processing them. If they react quickly enough, they can type their connection's defined _flush_ command; when the server first reads that command from the network, it immediately and completely flushes any as-yet unprocessed input from that user, printing a message to the user describing just which lines of input were discarded, if any.

> _Fine point:_ The flush command is handled very early in the server's processing of a line of input, before the line is entered into the task queue for the connection and well before it is parsed into words like other commands. For this reason, it must be typed exactly as it was defined, alone on the line, without quotation marks, and without any spaces before or after it.

When a connection is first accepted by the server, it is given an initial flush command setting taken from the current default. This initial setting can be changed later using the <span class="cmd">set_connection_option()</span> command.

By default, each connection is initially given <span class="cmd">.flush</span> as its flush command. If the property <span class="cmd">$server_options.default_flush_command</span> exists, then its value overrides this default. If <span class="cmd">$server_options.default_flush_command</span> is a non-empty string, then that string is the flush command for all new connections; otherwise, new connections are initially given no flush command at all.

### [Initial Punctuation in Commands](ProgrammersManual_toc.html#TOC61)

The server interprets command lines that begin with any of the following characters specially:

<pre>"        :        ;
</pre>

Before processing the command, the initial punctuation character is replaced by the corresponding word below, followed by a space:

<pre>say      emote    eval
</pre>

For example, the command line

<pre>"Hello, there.
</pre>

is transformed into

<pre>say Hello, there.
</pre>

before parsing.

## [Server Assumptions About the Database](ProgrammersManual_toc.html#TOC62)

There are a small number of circumstances under which the server directly and specifically accesses a particular verb or property in the database. This section gives a complete list of such circumstances.

### [Server Options Set in the Database](ProgrammersManual_toc.html#TOC63)

Many optional behaviors of the server can be controlled from within the database by creating the property <span class="cmd">#0.server_options</span> (also known as <span class="cmd">$server_options</span>), assigning as its value a valid object number, and then defining various properties on that object. At a number of times, the server checks for whether the property <span class="cmd">$server_options</span> exists and has an object number as its value. If so, then the server looks for a variety of other properties on that <span class="cmd">$server_options</span> object and, if they exist, uses their values to control how the server operates.

The specific properties searched for are each described in the appropriate section below, but here is a brief list of all of the relevant properties for ease of reference:

<dl compact="">

<dt><span class="cmd">bg_seconds</span></dt>

<dd>The number of seconds allotted to background tasks.</dd>

<dt><span class="cmd">bg_ticks</span></dt>

<dd>The number of ticks allotted to background tasks.</dd>

<dt><span class="cmd">connect_timeout</span></dt>

<dd>The maximum number of seconds to allow an un-logged-in in-bound connection to remain open.</dd>

<dt><span class="cmd">default_flush_command</span></dt>

<dd>The initial setting of each new connection's flush command.</dd>

<dt><span class="cmd">fg_seconds</span></dt>

<dd>The number of seconds allotted to foreground tasks.</dd>

<dt><span class="cmd">fg_ticks</span></dt>

<dd>The number of ticks allotted to foreground tasks.</dd>

<dt><span class="cmd">max_stack_depth</span></dt>

<dd>The maximum number of levels of nested verb calls.</dd>

<dt><span class="cmd">name_lookup_timeout</span></dt>

<dd>The maximum number of seconds to wait for a network hostname/address lookup.</dd>

<dt><span class="cmd">outbound_connect_timeout</span></dt>

<dd>The maximum number of seconds to wait for an outbound network connection to successfully open.</dd>

<dt><span class="cmd">protect_<var>property</var></span></dt>

<dd>Restrict reading of built-in <var>property</var> to wizards.</dd>

<dt><span class="cmd">protect_<var>function</var></span></dt>

<dd>Restrict use of built-in <var>function</var> to wizards.</dd>

<dt><span class="cmd">support_numeric_verbname_strings</span></dt>

<dd>Enables use of an obsolete verb-naming mechanism.</dd>

</dl>

### [Server Messages Set in the Database](ProgrammersManual_toc.html#TOC64)

There are a number of circumstances under which the server itself generates messages on network connections. Most of these can be customized or even eliminated from within the database. In each such case, a property on <span class="cmd">$server_options</span> is checked at the time the message would be printed. If the property does not exist, a default message is printed. If the property exists and its value is not a string or a list containing strings, then no message is printed at all. Otherwise, the string(s) are printed in place of the default message, one string per line. None of these messages are ever printed on an outbound network connection created by the function <span class="cmd">open_network_connection()</span>.

The following list covers all of the customizable messages, showing for each the name of the relevant property on <span class="cmd">$server_options</span>, the default message, and the circumstances under which the message is printed:

<dl compact="">

<dt><span class="cmd">boot_msg = "*** Disconnected ***"</span></dt>

<dd>The function <span class="cmd">boot_player()</span> was called on this connection.</dd>

<dt><span class="cmd">connect_msg = "*** Connected ***"</span></dt>

<dd>The user object that just logged in on this connection existed before <span class="cmd">$do_login_command()</span> was called.</dd>

<dt><span class="cmd">create_msg = "*** Created ***"</span></dt>

<dd>The user object that just logged in on this connection did not exist before <span class="cmd">$do_login_command()</span> was called.</dd>

<dt><span class="cmd">recycle_msg = "*** Recycled ***"</span></dt>

<dd>The logged-in user of this connection has been recycled or renumbered (via the renumber() function).</dd>

<dt><span class="cmd">redirect_from_msg = "*** Redirecting connection to new port ***"</span></dt>

<dd>The logged-in user of this connection has just logged in on some other connection.</dd>

<dt><span class="cmd">redirect_to_msg = "*** Redirecting old connection to this port ***"</span></dt>

<dd>The user who just logged in on this connection was already logged in on some other connection.</dd>

<dt><span class="cmd">server_full_msg</span></dt>

<dd>Default:

<pre>*** Sorry, but the server cannot accept any more connections right now.
*** Please try again later.
</pre>

This connection arrived when the server really couldn't accept any more connections, due to running out of a critical operating system resource.</dd>

<dt><span class="cmd">timeout_msg = "*** Timed-out waiting for login. ***"</span></dt>

<dd>This in-bound network connection was idle and un-logged-in for at least <span class="cmd">CONNECT_TIMEOUT</span> seconds (as defined in the file <span class="cmd">options.h</span> when the server was compiled).</dd>

</dl>

> _Fine point:_ If the network connection in question was received at a listening point (established by the <span class="cmd">listen()</span> function) handled by an object <var>obj</var> other than <span class="cmd">#0</span>, then system messages for that connection are looked for on <span class="cmd"><var>obj</var>.server_options</span>; if that property does not exist, then <span class="cmd">$server_options</span> is used instead.

### [Checkpointing the Database](ProgrammersManual_toc.html#TOC65)

The server maintains the entire MOO database in main memory, not on disk. It is therefore necessary for it to dump the database to disk if it is to persist beyond the lifetime of any particular server execution. The server is careful to dump the database just before shutting down, of course, but it is also prudent for it to do so at regular intervals, just in case something untoward happens.

To determine how often to make these _checkpoints_ of the database, the server consults the value of <span class="cmd">#0.dump_interval</span>. If it exists and its value is an integer greater than or equal to 60, then it is taken as the number of seconds to wait between checkpoints; otherwise, the server makes a new checkpoint every 3600 seconds (one hour). If the value of <span class="cmd">#0.dump_interval</span> implies that the next checkpoint should be scheduled at a time after 3:14:07 a.m. on Tuesday, January 19, 2038, then the server instead uses the default value of 3600 seconds in the future.

The decision about how long to wait between checkpoints is made again immediately after each one begins. Thus, changes to <span class="cmd">#0.dump_interval</span> will take effect after the next checkpoint happens.

Whenever the server begins to make a checkpoint, it makes the following verb call:

<pre>$checkpoint_started()
</pre>

When the checkpointing process is complete, the server makes the following verb call:

<pre>$checkpoint_finished(<var>success</var>)
</pre>

where <var>success</var> is true if and only if the checkpoint was successfully written on the disk. Checkpointing can fail for a number of reasons, usually due to exhaustion of various operating system resources such as virtual memory or disk space. It is not an error if either of these verbs does not exist; the corresponding call is simply skipped.

### [Accepting and Initiating Network Connections](ProgrammersManual_toc.html#TOC66)

When the server first accepts a new, incoming network connection, it is given the low-level network address of computer on the other end. It immediately attempts to convert this address into the human-readable host name that will be entered in the server log and returned by the <span class="cmd">connection_name()</span> function. This conversion can, for the TCP/IP networking configurations, involve a certain amount of communication with remote name servers, which can take quite a long time and/or fail entirely. While the server is doing this conversion, it is not doing anything else at all; in particular, it it not responding to user commands or executing MOO tasks.

By default, the server will wait no more than 5 seconds for such a name lookup to succeed; after that, it behaves as if the conversion had failed, using instead a printable representation of the low-level address. If the property <span class="cmd">name_lookup_timeout</span> exists on <span class="cmd">$server_options</span> and has an integer as its value, that integer is used instead as the timeout interval.

When the <span class="cmd">open_network_connection()</span> function is used, the server must again do a conversion, this time from the host name given as an argument into the low-level address necessary for actually opening the connection. This conversion is subject to the same timeout as in the in-bound case; if the conversion does not succeed before the timeout expires, the connection attempt is aborted and <span class="cmd">open_network_connection()</span> raises <span class="cmd">E_QUOTA</span>.

After a successful conversion, though, the server must still wait for the actual connection to be accepted by the remote computer. As before, this can take a long time during which the server is again doing nothing else. Also as before, the server will by default wait no more than 5 seconds for the connection attempt to succeed; if the timeout expires, <span class="cmd">open_network_connection()</span> again raises <span class="cmd">E_QUOTA</span>. This default timeout interval can also be overridden from within the database, by defining the property <span class="cmd">outbound_connect_timeout</span> on <span class="cmd">$server_options</span> with an integer as its value.

### [Associating Network Connections with Players](ProgrammersManual_toc.html#TOC67)

When a network connection is first made to the MOO, it is identified by a unique, negative object number. Such a connection is said to be _un-logged-in_ and is not yet associated with any MOO player object.

Each line of input on an un-logged-in connection is first parsed into words in the usual way (see the chapter on command parsing for details) and then these words are passed as the arguments in a call to the verb <span class="cmd">$do_login_command()</span>. For example, the input line

<pre>connect Munchkin frebblebit
</pre>

would result in the following call being made:

<pre>$do_login_command("connect", "Munchkin", "frebblebit")
</pre>

In that call, the variable <span class="cmd">player</span> will have as its value the negative object number associated with the appropriate network connection. The functions <span class="cmd">notify()</span> and <span class="cmd">boot_player()</span> can be used with such object numbers to send output to and disconnect un-logged-in connections. Also, the variable <span class="cmd">argstr</span> will have as its value the unparsed command line as received on the network connection.

If <span class="cmd">$do_login_command()</span> returns a valid player object and the connection is still open, then the connection is considered to have _logged into_ that player. The server then makes one of the following verbs calls, depending on the player object that was returned:

<pre>$user_created(<var>player</var>)
$user_connected(<var>player</var>)
$user_reconnected(<var>player</var>)
</pre>

The first of these is used if the returned object number is greater than the value returned by the <span class="cmd">max_object()</span> function before <span class="cmd">$do_login_command()</span> was invoked, that is, it is called if the returned object appears to have been freshly created. If this is not the case, then one of the other two verb calls is used. The <span class="cmd">$user_connected()</span> call is used if there was no existing active connection for the returned player object. Otherwise, the <span class="cmd">$user_reconnected()</span> call is used instead.

> _Fine point:_ If a user reconnects and the user's old and new connections are on two different listening points being handled by different objects (see the description of the <span class="cmd">listen()</span> function for more details), then <span class="cmd">user_client_disconnected</span> is called for the old connection and <span class="cmd">user_connected</span> for the new one.

If an in-bound network connection does not successfully log in within a certain period of time, the server will automatically shut down the connection, thereby freeing up the resources associated with maintaining it. Let <var>L</var> be the object handling the listening point on which the connection was received (or <span class="cmd">#0</span> if the connection came in on the initial listening point). To discover the timeout period, the server checks on <span class="cmd"><var>L</var>.server_options</span> or, if it doesn't exist, on <span class="cmd">$server_options</span> for a <span class="cmd">connect_timeout</span> property. If one is found and its value is a positive integer, then that's the number of seconds the server will use for the timeout period. If the <span class="cmd">connect_timeout</span> property exists but its value isn't a positive integer, then there is no timeout at all. If the property doesn't exist, then the default timeout is 300 seconds.

When any network connection (even an un-logged-in or outbound one) is terminated, by either the server or the client, then one of the following two verb calls is made:

<pre>$user_disconnected(<var>player</var>)
$user_client_disconnected(<var>player</var>)
</pre>

The first is used if the disconnection is due to actions taken by the server (e.g., a use of the <span class="cmd">boot_player()</span> function or the un-logged-in timeout described above) and the second if the disconnection was initiated by the client side.

It is not an error if any of these five verbs do not exist; the corresponding call is simply skipped.

> **Note**: Only one network connection can be controlling a given player object at a given time; should a second connection attempt to log in as that player, the first connection is unceremoniously closed (and <span class="cmd">$user_reconnected()</span> called, as described above). This makes it easy to recover from various kinds of network problems that leave connections open but inaccessible.

When the network connection is first established, the null command is automatically entered by the server, resulting in an initial call to <span class="cmd">$do_login_command()</span> with no arguments. This signal can be used by the verb to print out a welcome message, for example.

> **Warning**: If there is no <span class="cmd">$do_login_command()</span> verb defined, then lines of input from un-logged-in connections are simply discarded. Thus, it is _necessary_ for any database to include a suitable definition for this verb.

### [Out-of-Band Commands](ProgrammersManual_toc.html#TOC68)

It is possible to compile the server with an option defining an _out-of-band prefix_ for commands. This is a string that the server will check for at the beginning of every line of input from players, regardless of whether or not those players are logged in and regardless of whether or not reading tasks are waiting for input from those players. If a given line of input begins with the defined out-of-band prefix (leading spaces, if any, are _not_ stripped before testing), then it is not treated as a normal command or as input to any reading task. Instead, the line is parsed into a list of words in the usual way and those words are given as the arguments in a call to <span class="cmd">$do_out_of_band_command()</span>. For example, if the out-of-band prefix were defined to be <span class="cmd">#$#</span>, then the line of input

<pre>#$# client-type fancy
</pre>

would result in the following call being made in a new server task:

<pre>$do_out_of_band_command("#$#", "client-type", "fancy")
</pre>

During the call to <span class="cmd">$do_out_of_band_command()</span>, the variable <span class="cmd">player</span> is set to the object number representing the player associated with the connection from which the input line came. Of course, if that connection has not yet logged in, the object number will be negative. Also, the variable <span class="cmd">argstr</span> will have as its value the unparsed input line as received on the network connection.

Out-of-band commands are intended for use by fancy client programs that may generate asynchronous _events_ of which the server must be notified. Since the client cannot, in general, know the state of the player's connection (logged-in or not, reading task or not), out-of-band commands provide the only reliable client-to-server communications channel.

### [The First Tasks Run By the Server](ProgrammersManual_toc.html#TOC69)

Whenever the server is booted, there are a few tasks it runs right at the beginning, before accepting connections or getting the value of <span class="cmd">#0.dump_interval</span> to schedule the first checkpoint (see below for more information on checkpoint scheduling).

First, the server calls <span class="cmd">$user_disconnected()</span> once for each user who was connected at the time the database file was written; this allows for any cleaning up that's usually done when users disconnect (e.g., moving their player objects back to some `home' location, etc.).

Next, it checks for the existence of the verb <span class="cmd">$server_started()</span>. If there is such a verb, then the server runs a task invoking that verb with no arguments and with <span class="cmd">player</span> equal to <span class="cmd">#-1</span>. This is useful for carefully scheduling checkpoints and for re-initializing any state that is not properly represented in the database file (e.g., re-opening certain outbound network connections, clearing out certain tables, etc.).

### [Controlling the Execution of Tasks](ProgrammersManual_toc.html#TOC70)

As described earlier, in the section describing MOO tasks, the server places limits on the number of seconds for which any task may run continuously and the number of "ticks," or low-level operations, any task may execute in one unbroken period. By default, foreground tasks may use 30,000 ticks and five seconds, and background tasks may use 15,000 ticks and three seconds. These defaults can be overridden from within the database by defining any or all of the following properties on <span class="cmd">$server_options</span> and giving them integer values:

<dl compact="">

<dt><span class="cmd">bg_seconds</span></dt>

<dd>The number of seconds allotted to background tasks.</dd>

<dt><span class="cmd">bg_ticks</span></dt>

<dd>The number of ticks allotted to background tasks.</dd>

<dt><span class="cmd">fg_seconds</span></dt>

<dd>The number of seconds allotted to foreground tasks.</dd>

<dt><span class="cmd">fg_ticks</span></dt>

<dd>The number of ticks allotted to foreground tasks.</dd>

</dl>

The server ignores the values of <span class="cmd">fg_ticks</span> and <span class="cmd">bg_ticks</span> if they are less than 100 and similarly ignores <span class="cmd">fg_seconds</span> and <span class="cmd">bg_seconds</span> if their values are less than 1\. This may help prevent utter disaster should you accidentally give them uselessly-small values.

Recall that command tasks and server tasks are deemed _foreground_ tasks, while forked, suspended, and reading tasks are defined as _background_ tasks. The settings of these variables take effect only at the beginning of execution or upon resumption of execution after suspending or reading.

The server also places a limit on the number of levels of nested verb calls, raising <span class="cmd">E_MAXREC</span> from a verb-call expression if the limit is exceeded. The limit is 50 levels by default, but this can be increased from within the database by defining the <span class="cmd">max_stack_depth</span> property on <span class="cmd">$server_options</span> and giving it an integer value greater than 50\. The maximum stack depth for any task is set at the time that task is created and cannot be changed thereafter. This implies that suspended tasks, even after being saved in and restored from the DB, are not affected by later changes to $server_options.max_stack_depth.

Finally, the server can place a limit on the number of forked or suspended tasks any player can have queued at a given time. Each time a <span class="cmd">fork</span> statement or a call to <span class="cmd">suspend()</span> is executed in some verb, the server checks for a property named <span class="cmd">queued_task_limit</span> on the programmer. If that property exists and its value is a non-negative integer, then that integer is the limit. Otherwise, if <span class="cmd">$server_options.queued_task_limit</span> exists and its value is a non-negative integer, then that's the limit. Otherwise, there is no limit. If the programmer already has a number of queued tasks that is greater than or equal to the limit, <span class="cmd">E_QUOTA</span> is raised instead of either forking or suspending. Reading tasks are affected by the queued-task limit.

### [Controlling the Handling of Aborted Tasks](ProgrammersManual_toc.html#TOC71)

The server will abort the execution of tasks for either of two reasons:

1.  an error was raised within the task but not caught, or
2.  the task exceeded the limits on ticks and/or seconds.

In each case, after aborting the task, the server attempts to call a particular _handler verb_ within the database to allow code there to handle this mishap in some appropriate way. If this verb call suspends or returns a true value, then it is considered to have handled the situation completely and no further processing will be done by the server. On the other hand, if the handler verb does not exist, or if the call either returns a false value without suspending or itself is aborted, the server takes matters into its own hands.

First, an error message and a MOO verb-call stack _traceback_ are printed to the player who typed the command that created the original aborted task, explaining why the task was aborted and where in the task the problem occurred. Then, if the call to the handler verb was itself aborted, a second error message and traceback are printed, describing that problem as well. Note that if the handler-verb call itself is aborted, no further `nested' handler calls are made; this policy prevents what might otherwise be quite a vicious little cycle.

The specific handler verb, and the set of arguments it is passed, differs for the two causes of aborted tasks.

If an error is raised and not caught, then the verb-call

<pre>$handle_uncaught_error(<var>code</var>, <var>msg</var>, <var>value</var>, <var>traceback</var>, <var>formatted</var>)
</pre>

is made, where <var>code</var>, <var>msg</var>, <var>value</var>, and <var>traceback</var> are the values that would have been passed to a handler in a <span class="cmd">try</span>-<span class="cmd">except</span> statement and <var>formatted</var> is a list of strings being the lines of error and traceback output that will be printed to the player if <span class="cmd">$handle_uncaught_error</span> returns false without suspending.

If a task runs out of ticks or seconds, then the verb-call

<pre>$handle_task_timeout(<var>resource</var>, <var>traceback</var>, <var>formatted</var>)
</pre>

is made, where <var>resource</var> is the appropriate one of the strings <span class="cmd">"ticks"</span> or <span class="cmd">"seconds"</span>, and <var>traceback</var> and <var>formatted</var> are as above.

### [Matching in Command Parsing](ProgrammersManual_toc.html#TOC72)

In the process of matching the direct and indirect object strings in a command to actual objects, the server uses the value of the <span class="cmd">aliases</span> property, if any, on each object in the contents of the player and the player's location. For complete details, see the chapter on command parsing.

### [Restricting Access to Built-in Properties and Functions](ProgrammersManual_toc.html#TOC73)

Whenever verb code attempts to read the value of a built-in property <var>prop</var> on any object, the server checks to see if the property <span class="cmd">$server_options.protect_<var>prop</var></span> exists and has a true value. If so, then <span class="cmd">E_PERM</span> is raised if the programmer is not a wizard.

Whenever verb code calls a built-in function <span class="cmd"><var>func</var>()</span> and the caller is not the object <span class="cmd">#0</span>, the server checks to see if the property <span class="cmd">$server_options.protect_<var>func</var></span> exists and has a true value. If so, then the server next checks to see if the verb <span class="cmd">$bf_<var>func</var>()</span> exists; if that verb exists, then the server calls it _instead_ of the built-in function, returning or raising whatever that verb returns or raises. If the <span class="cmd">$bf_<var>func</var>()</span> does not exist and the programmer is not a wizard, then the server immediately raises <span class="cmd">E_PERM</span>, _without_ actually calling the function. Otherwise (if the caller is <span class="cmd">#0</span>, if <span class="cmd">$server_options.protect_<var>func</var></span> either doesn't exist or has a false value, or if <span class="cmd">$bf_<var>func</var>()</span> exists but the programmer is a wizard), then the built-in function is called normally.

### [Creating and Recycling Objects](ProgrammersManual_toc.html#TOC74)

Whenever the <span class="cmd">create()</span> function is used to create a new object, that object's <span class="cmd">initialize</span> verb, if any, is called with no arguments. The call is simply skipped if no such verb is defined on the object.

Symmetrically, just before the <span class="cmd">recycle()</span> function actually destroys an object, the object's <span class="cmd">recycle</span> verb, if any, is called with no arguments. Again, the call is simply skipped if no such verb is defined on the object.

Both <span class="cmd">create()</span> and <span class="cmd">recycle()</span> check for the existence of an <span class="cmd">ownership_quota</span> property on the owner of the newly-created or -destroyed object. If such a property exists and its value is an integer, then it is treated as a _quota_ on object ownership. Otherwise, the following two paragraphs do not apply.

The <span class="cmd">create()</span> function checks whether or not the quota is positive; if so, it is reduced by one and stored back into the <span class="cmd">ownership_quota</span> property on the owner. If the quota is zero or negative, the quota is considered to be exhausted and <span class="cmd">create()</span> raises <span class="cmd">E_QUOTA</span>.

The <span class="cmd">recycle()</span> function increases the quota by one and stores it back into the <span class="cmd">ownership_quota</span> property on the owner.

### [Object Movement](ProgrammersManual_toc.html#TOC75)

During evaluation of a call to the <span class="cmd">move()</span> function, the server can make calls on the <span class="cmd">accept</span> and <span class="cmd">enterfunc</span> verbs defined on the destination of the move and on the <span class="cmd">exitfunc</span> verb defined on the source. The rules and circumstances are somewhat complicated and are given in detail in the description of the <span class="cmd">move()</span> function.

### [Temporarily Enabling Obsolete Server Features](ProgrammersManual_toc.html#TOC76)

If the property <span class="cmd">$server_options.support_numeric_verbname_strings</span> exists and has a true value, then the server supports a obsolete mechanism for less ambiguously referring to specific verbs in various built-in functions. For more details, see the discussion given just following the description of the <span class="cmd">verbs()</span> function.