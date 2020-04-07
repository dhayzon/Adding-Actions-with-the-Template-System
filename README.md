Adding Actions with the Template System
================================================================================
When developing themes for SMF, there are times when you'll want an extra page.
This might be a "copyright" page or a "confirmation" page for something.

The template system includes ways of including actions, in your index template
file.  This functionality is called "catching" or "wrapping" an action.  Please
note that this will only work for actions NOT otherwise defined - you can't
define a new "notify" action because one already exists.


Wrapping Actions
--------------------------------------------------------------------------------
The "wrapping" method is better for when you want undefined actions to go to a
certain page, like a 404, or when you want to handle a small number of actions.
If you want to work with a large set of actions, not just a single one, you
probably should go with the "catching" method below.

First, in your index template, look at the init sub template.  You can find it
by searching for "template_init".  This is a special sub template run during
SMF's initialization before some of the context variables have been filled.

It starts out without much in it.  Look for the closing curly brace (}) on its
own line - that's the end of it.  Right above that, add:
	$settings['catch_action'] = array('sub_template' => 'action_wrap');

Congratulations - you've wrapped the action!  But what does that MEAN!?  Don't
worry, it just means that now the action_wrap sub template will be shown
whenever someone goes to an action otherwise not defined.

So, to use this, find that closing curly brace again (the }) and add right after
it the following:
function template_action_wrap()
{
	global $context, $settings, $options, $scripturl, $txt;
}

Now, see those tricky curly braces again?  Inside you can add your 404 page.
As an example, you might have:

function template_action_wrap()
{
	global $context, $settings, $options, $scripturl, $txt;

	echo '
		<h1>404 Error!</h1>

		<p>Sorry, the page you were looking for does not exist.</p>
		Please go <a href="javascript:history.go(-1);">back</a> or
		view the <a href="', $scripturl, '">forum index</a>.';
}

If you don't want a 404 error there, you can add something to delegate the
action.  Delegating is when the action_wrap sub template sends stuff over
to another template.  You can also just show different html depending on
the action.

For example, for a copyright page you might simply:

function template_action_wrap()
{
	global $context, $settings, $options, $scripturl, $txt;

	if (@$_REQUEST['action'] == 'copyright')
		echo '
		<h1>Copyright</h1>

		This theme copyright &copy;2004 by Unknown W. Brackets.<br />
		Smileys copyright by Andrea H.<br />
		Forum software copyright 2001-2004 by Lewis Media.';
	else
		echo '
		<h1>404 Error!</h1>

		<p>Sorry, the page you were looking for does not exist.</p>
		Please go <a href="javascript:history.go(-1);">back</a> or
		view the <a href="', $scripturl, '">forum index</a>.';
}

To "delegate" you might do something like the following:

function template_action_wrap()
{
	global $context, $settings, $options, $scripturl, $txt;

	if (@$_REQUEST['action'] == 'copyright')
		template_copyright();
	else
		template_four_oh_four();
}

And then define those sub templates below.


Catching Actions
--------------------------------------------------------------------------------
Catching actions isn't very hard, and can be used to do many actions more
efficiently.  It differs from "wrapping" actions only subtly (just as the names
are only subtly different...) in that the action is validated a bit more.

First, in your index template, look at the init sub template.  You can find it
by searching for "template_init".  This is a special sub template run during
SMF's initialization before some of the context variables have been filled.

It starts out without much in it.  Look for the closing curly brace (}) on its
own line - that's the end of it.  Just above that, you might add:

	if (@$_REQUEST['action'] == 'copyright')
		$settings['catch_action'] = array('template' => 'Copyright');

This means that if someone goes to the ?action=copyright action, the Copyright
template will be loaded along with the main sub template.  Pretty cool, huh?

You can do this more and more, like so:

	if (@$_REQUEST['action'] == 'copyright')
		$settings['catch_action'] = array('template' => 'Copyright');
	if (@$_REQUEST['action'] == 'magicllama')
		$settings['catch_action'] = array('template' => 'MagicLlama');

Wrapping and Catching
--------------------------------------------------------------------------------
You can do both of the above by following a simple example:

	$settings['catch_action'] = array('template' => 'Four-Oh-Four');
	if (@$_REQUEST['action'] == 'copyright')
		$settings['catch_action'] = array('template' => 'Copyright');
	if (@$_REQUEST['action'] == 'magicllama')
		$settings['catch_action'] = array('template' => 'MagicLlama');

And it will go to the Four-Oh-Four template if the action is not copyright AND
not magicllama.


What's Available to Load
--------------------------------------------------------------------------------
You might have noticed that this is all done using the 'catch_action' setting.
This setting can have many different options:

	template:
		The template to load when the action is "caught".  This also loads the
		corresponding language file at the same time.

	layers:
		An array of layers.  For example, you might use array('main', 'admin')
		to show the main and admin layers - just like the admin center.

	function:
		The function to call.  If this is set, the sub_template option will be
		ignored completely.

	filename:
		The name of the file to load from under the Sources directory.  This
		only has affect when it is set and 'function' is set.

	sub_template:
		The sub template to load - doesn't work with function/filename.

You can combine these like so:

	$settings['catch_action'] = array(
		'template' => 'CoolTemplate',
		'layers' => array('main', 'coolness'),
		'sub_template' => 'coolinate'
	);

Or like this, using a file from Sources: (to automatically go to your profile.)

	$settings['catch_action'] = array(
		'filename' => 'Profile.php',
		'function' => 'ModifyProfile'
	);

If you have any questions or suggestions on how you think this should change,
just ask!
Simple Machines
