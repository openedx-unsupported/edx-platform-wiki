### Debugging the devstack image with PyCharm professional edition

Debugging with devstack is more complex than in a direct local development environment, as the source code lives on your local machine but the code is executing within the Vagrant image. One solution is to use [PyCharm Professional Edition](http://www.jetbrains.com/pycharm/) from JetBrains. It is able to debug devstack using its concept of [remote Python interpreters](http://blog.jetbrains.com/pycharm/2013/03/how-pycharm-helps-you-with-remote-development/). Similar setups should also work for any IDE with remote interpreter.

Here are the steps to use PyCharm on MacOS (other Unix environments should be similar). 

* First, start up a terminal and give the 'edxapp' user a password
```
  vagrant ssh
  sudo passwd edxapp
  [type in a password of your choice]
```
* Next, create the .pycharm_helpers directory where PyCharm will store its remote debugging code
```
  sudo -
  su edxapp
  mkdir .pycharm_helpers
```
* Create a PyCharm project for devstack (if you don't have one already):
  * Select "New Project..." from the "File" menu
  * Select your devstack folder as the "Location"
  * Choose "Empty project" for "Project type" (note: don't choose "Django project" as that can't currently be interpreted remotely)
  * Say "Yes" to the question about using existing sources for the project
* To make PyCharm's editor traverse imports in edx-platform python modules, go to "File > Settings... > Project Structure" and mark the following directories as "Source Folders":
  * lms/djangoapps
  * lms/lib
  * cms/djangoapps
  * cms/lib
  * common/djangoapps
  * common/lib
* You can also mark the following symlinks to be "Excluded" so that PyCharm doesn't see multiple copies of the same files:
  * cms/static/xmodule_js
  * lms/static/xmodule_js

### Create a PyCharm remote interpreter (PyCharm v3.4+)
* Select "Preferences..." from the Apple menu
* Expand "Project Interpreter" under "Project Settings"
* Click the icon in the upper right corner of the dialog box--it looks like it may be a gear or a star.
* In the popup dialog that appears, click on "Add Remote"
* When the remote dialog appears, select the "SSH Credentials" radio button.
* Specify the following configurations:
![PyCharm 3.4+ conf](https://1786529bf2dfcc9a4fc2736524bc8aea4a66cc50.googledrive.com/host/0BxQlaq542xl2V182QTM4ZF9kZlU/pycharm_remote_conf.png)
* PyCharm will automatically detect the helper folder and upload the debugging files. The final configuration should look like this:
![PyCharm 3.4+ conf] (https://1786529bf2dfcc9a4fc2736524bc8aea4a66cc50.googledrive.com/host/0BxQlaq542xl2V182QTM4ZF9kZlU/pycharm_conf.jpg)
* After that, you should verify that PyCharm has copied all debugging materials into the ```.pycharm_helpers``` folder by ssh-ing into the vagrant instance and navigating to the helper folder. The directory content should look similar to this:
![PyCharm helper](https://1786529bf2dfcc9a4fc2736524bc8aea4a66cc50.googledrive.com/host/0BxQlaq542xl2V182QTM4ZF9kZlU/pycharm_helpers.png)
 

### Create Debug Configuration for LMS
After the PyCharm remote interpreter is configured we are ready to debug devstack.

* Create a debug configuration for LMS server
  * Open the file 'edx-platform/manage.py' in your devstack folder
  * Right-click on the file and choose "Create 'manage'..." to create a debug configuration
  * Specify the settings for the new configuration:
    * Change the name to something more memorable, e.g. "LMS".
    * Specify "/edx/app/edxapp/venvs/edxapp/bin/paver" for "Script".
    * Specify "devstack lms" for "Script parameters".
    * click to edit the "Environment variables" property
      * Click the "+" button to add a new variable
      * add all variables included in /edx/app/edxapp/edxapp_env 
        * LANG="en_US.UTF-8"
        * SKIP_WS_MIGRATIONS="1"
        * GEM_PATH="/edx/app/edxapp/.gem"
        * NO_PREREQ_INSTALL="1"
        * PATH="/edx/app/edxapp/venvs/edxapp/bin:/edx/app/edxapp/edx-platform/bin:/edx/app/edxapp/.rbenv/bin:/edx/app/edxapp/.rbenv/shims:/edx/app/edxapp/.gem/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
        * GEM_HOME="/edx/app/edxapp/.gem"
        * RBENV_ROOT="/edx/app/edxapp/.rbenv"
    * Choose the remote interpreter (usually named as "Remote Python 2.7.3 (ssh://edxapp.127.0.0.1:2222/...)").
    * Specify "/edx/app/edxapp/edx-platform" for "Working directory".
    * Click to edit the "Path mappings" property
      * Click the "+" button to add a new mapping
      * Specify the full local path to "edx-platform" for "Local path". This should be the edx-platform directly beneath your devstack folder, e.g. "/Users/andya/devstack/edx-platform"  (Be sure to specify "Users" rather than the shorthand "~" here.)
      * Specify "/edx/app/edxapp/edx-platform" for "Remote path".
      * Click "OK" to save the mappings.
    * Deselect "Add content roots to PYTHONPATH".
    * Deselect "Add source roots to PYTHONPATH".
  * Click "OK" to save the new debug configuration.
  * Your LMS server configuration should look something similar to:
![LMS server configuration]
(https://1786529bf2dfcc9a4fc2736524bc8aea4a66cc50.googledrive.com/host/0BxQlaq542xl2V182QTM4ZF9kZlU/LMS_server.png)

* Debug the new "LMS" configuration
  * Choose "Run > Debug..."
  * Specify "LMS" and the Django instance should be started up.
  * You should now be able to set breakpoints and hit them.

### Create a debug configuration for Studio (CMS) 
* The Studio configuration is very similar to LMS, so we need to clone the LMS configuration and adjust a few parameters. 
  * Choose "Run > Edit Configurations..."
  * Select "LMS"
  * Click the "Copy configuration" button (next to the "-" button)
  * Change the name to "Studio"
  * Change the "Script parameters" to ```devstack studio```
  * Click "OK" to save the new configuration
* If you have XQuartz installed, you can test the server directly by going logging in as edxapp and type ```firefox```.

### Testing your changes

See the [[Test Engineering FAQ]] for all your questions about testing the edX platform.

### Debugging edX platform tests in PyCharm

Now with the remote interpreter, we can use PyCharm to debug edX platform tests (Python, JavaScript, Bok Choy, Acceptance) kicked off via paver command. The process is fairly simple. Since the test suites can be run using the ```paver``` command with different parameters, we just need to supply the appropriate paver task and parameters.

### Setting up JavaScript Test Configuration
* Go to Run -> Edit Configurations -> Add New Configuration (usually a +sign on the left).
* Script: ```/edx/app/edxapp/venvs/edxapp/bin/paver```.
* Script parameters: ```test_js```. 
* Choose the remote interpreter (usually named as "Remote Python 2.7.3 (ssh://edxapp.127.0.0.1:2222/...)").
* Working directory: ```/Users/[username]/devstack/edx-platform```
* Path mappings: ```/Users/[username]/devstack/edx-platform=/edx/app/edxapp/edx-platform```

### Setting up Python Test Configuration
* Go to Run -> Edit Configurations -> Add New Configuration (usually a +sign on the left).
* Script: ```/edx/app/edxapp/venvs/edxapp/bin/paver```.
* Script parameters: ```test_python```. 
* Choose the remote interpreter (usually named as "Remote Python 2.7.3 (ssh://edxapp.127.0.0.1:2222/...)").
* Working directory: ```/Users/[username]/devstack/edx-platform```
* Path mappings: ```/Users/[username]/devstack/edx-platform=/edx/app/edxapp/edx-platform```

### Setting up Bokchoy Test Configuration
![Bokchoy Test Configuration]
(https://1786529bf2dfcc9a4fc2736524bc8aea4a66cc50.googledrive.com/host/0BxQlaq542xl2V182QTM4ZF9kZlU/bokchoy_server.png)
* Go to Run -> Edit Configurations -> Add New Configuration (usually a +sign on the left).
* Script: ```/edx/app/edxapp/venvs/edxapp/bin/paver```.
* Script parameters: ```test_bokchoy``` or ```test_bokchoy --fasttest``` (if you have already compiled the assets). 
* Choose the remote interpreter (usually named as "Remote Python 2.7.3 (ssh://edxapp.127.0.0.1:2222/...)").
* Working directory: ```/Users/[username]/devstack/edx-platform```
* Path mappings: ```/Users/[username]/devstack/edx-platform=/edx/app/edxapp/edx-platform```

### Create a debug configuration for an edX common unit test          
* Choose "Run > Edit Configurations..."
* Select the "Studio" configuration.
* Click the "Copy configuration" button (next to the "-" button).
* Change the name to "Studio CommonTests".
* Change the script to ```/edx/app/edxapp/venvs/edxapp/bin/nosetests```
* Change the "Script parameters" to run the test:
  * e.g. ```common/lib/xmodule/xmodule/tests/test_resource_templates.py```

### Create a debug configuration for an edX CMS unit test
* Choose "Run > Edit Configurations..."
* Select the "Studio" configuration.
* Click the "Copy configuration" button (next to the "-" button).
* Change the name to "Studio Tests".
* Change the "Script parameters" to run the test:
  * e.g. ```cms --settings test test cms/djangoapps/contentstore/views/tests/test_helpers.py```
              
### Create a debug configuration for an edX acceptance test
* Choose "Run > Edit Configurations..."
* Select the "Studio" configuration.
* Click the "Copy configuration" button (next to the "-" button).
* Change the name to "Studio Acceptance Tests".
* Change the "Script parameters" to run the test:
  * e.g. ```cms --settings acceptance harvest --traceback --debug-mode --verbosity 2 --with-xunit --xunit-file /edx/app/edxapp/edx-platform/reports/acceptance/cms.xml cms/djangoapps/contentstore/features/problem-editor.feature```

### Visually debug your tests with XQuartz
You can setup your development environment such that you can visually interact with browsers and other GUIs in the vagrant machine from the host machine. To do this, you will need to install [XQuartz](http://xquartz.macosforge.org/landing/). For further information, see [Test Engineering FAQ](https://github.com/edx/edx-platform/wiki/Test-engineering-FAQ#im-working-with-devstack-and-want-to-debug-the-jasmine-or-acceptance-tests-in-the-browser-on-my-host-system-how-do-i-do-that).

### Integrate XQuartz into PyCharm
The following instructions are for Mac OS X but should work on any Unix-like environment.
* First, we need to enable XQuartz forwarding for all ssh agents. To do this, open ```/etc/ssh_config``` on your host system and uncomment ```ForwardX11 no``` and change to ```ForwardX11 yes```.
* Next open a terminal **inside** PyCharm (Tools => Open Terminal...") and type ```ssh edxapp@127.0.0.1 -p 2222```. After logging in as edxapp user, try to open Firefox by typing ```firefox``` into the terminal. XQuartz should fire up Firefox immediately. Keep this terminal **open**, DO NOT close it.
* Now we need to get the display port number from this terminal by ```echo $DISPLAY```. This depends on the number of existing ssh terminals. If this it the only one, it will be ```localhost:10.0```.
* Open a debug configuration (such as Bokchoy) and add DISPLAY to the environment variable with the port number from the previous step. The string ```DISPLAY=localhost:10.0``` should be inside the environment variable string.
![Display Port Environmental Variable]
(https://1786529bf2dfcc9a4fc2736524bc8aea4a66cc50.googledrive.com/host/0BxQlaq542xl2V182QTM4ZF9kZlU/display_port.png)
* Now we can start debugging Paver tests with browser windows popping up in XQuartz.
* Note: the port number (10.0, 11.0, etc, ...) changes dynamically with the number of open ssh clients. Therefore, you have to open and keep the ssh terminal inside Pycharm all the time.
* If the PyCharm debugger does not attach itself to the thread for some reasons, you can insert the break point manually:

```
from nose.tools import set_trace
set_trace()
```