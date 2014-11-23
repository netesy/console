console
=======

Create command line php applications using symfony/console.  
This is basically a fork of symfony/console which does things a little differently and adds some extra features.  
A lot of the features are useful when running commands in a background context (such as via crontab).  

[![Latest Stable Version](https://poser.pugx.org/duncan3dc/console/version.svg)](https://packagist.org/packages/duncan3dc/console)
[![Build Status](https://travis-ci.org/duncan3dc/console.svg?branch=master)](https://travis-ci.org/duncan3dc/console)



Loading Commands
----------------
Commands can be automatically created from classes (meaning you don't need to call setName() inside your command class) using the following criteria:
* Files/classes must be named using CamelCase and must end in "Command" (with files having the .php extension)
* Each uppercase character will be converted to lowercase and preceded by a hyphen
* Directories will represent namespaces and each separater will be replaced with a colon
Using the example below, the file src/commands/Category/Topic/RunCommand.php will create a command called category:topic:run
```php
$application->loadCommands("src/commands");
```
_Of course, they can still be added the [symfony way](http://symfony.com/doc/current/components/console/introduction.html)_



Output
------
We use [league/climate](http://climate.thephpleague.com/) for terminal output, whilst also maintaining support for the [symfony way](http://symfony.com/doc/current/components/console/introduction.html#coloring-the-output).  
So all of the following is possible:
```php
$output->blue()->out("Blue? Wow!");
$output->dump($complexArrayForCLImate);
$output->writeln("<error>I am a symfony/console error</error>");
```



Time Limit Commands
-------------------
Commands can limit how long they are run for, and end in a controlled way when the limit is reached.  
Inside your command's class you can call the timeout() method and pass the number of seconds your command should run for.  
```php
class LimitedCommand extends \duncan3dc\Console\Command
{
    protected function execute(InputInterface $input, OutputInterface $output)
    {
        while (true) {
            # If the command has been running for more than 10 minutes then end now
            if ($this->timeout(60 * 10)) {
                break;
            }
        }
    {
}
```
_This behaviour can be overridden by passing the ```--no-time-limit``` when running the application, this will cause the timeout() method to always return false_



Calling An Existing Command
---------------------------
The [symfony way](http://symfony.com/doc/current/components/console/introduction.html#calling-an-existing-command) of calling an existing command can be a little long-winded, with steps that seem unnecessary (eg, specifying the command name twice).  
The runCommand() method provides a simplified way of doing this (using the example from the symfony docs):
```php
$returnCode = $this->getApplication()->runCommand("demo:greet", [
    "name"      =>  "Fabien",
    "--yell"    =>  true,
], $input, $output);
```
_This also ensures any command event listeners that have been registered are called, which symfony does not do_



Command Locking
---------------
All commands are automatically locked to prevent the same command being run simultaneously on the same host.  
You can prevent particular commands from locking using the doNotLock() method:
```php
protected function configure()
{
    $this
        ->doNotLock()
        ->setDescription("This command can run as many times as it likes, whether the previous run has finished or not");
}
```
_When a command cannot run it will exit with status 201, represented by the class constant Application::STATUS_LOCKED_



Tab Completion
--------------
Tab completion is provided by [stecman/symfony-console-completion](https://github.com/stecman/symfony-console-completion) and instructions on setting it up for your application can be found in the README.md of that repository.  
