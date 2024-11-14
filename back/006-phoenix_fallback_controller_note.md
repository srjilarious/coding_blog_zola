+++
Tags = ["Phoenix", "Elixir"]
date = "2017-07-28T12:00:00+02:00"
title = "Tip for Phoenix 1.3 Fallback Controller error"
slug = "phoenix_fallback_controller_note"
Categories = ["Phoenix", "Elixir"]
+++

Just a quick note this time.  While working on my Phoenix 1.3 based project, I had a route that was failing and gave me an error about the fallback controller not having a matching clause.  Similar to (formatted excerpt):

    ** (exit) an exception was raised:
        ** (FunctionClauseError) no function clause 
        matching in MyProj.Web.FallbackController.call/2

My route looked to be setup correctly and I couldn't figure out why I was hitting the fallback controller at all.  It turned out the problem was one branch of my controller for that route didn't return a Plug.Conn and so my project fell back to the FallbackController.  Of course the Fallback controller didn't know what to do with my nil return value either, hence the error.

In any case, now knowing that that can occur, I was able to avoid spending too much time checking that my route was spelled correctly and such, instead looking to see that all the branches have a valid render call.  Hopefully this can save someone on the interwebs an hour of debugging.

<div id="commento"></div>
<script src="https://cdn.commento.io/js/commento.js"></script>
