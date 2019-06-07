# weatherStationUploader v4.0.0
Share your openHAB weather sensors data with the world!  It’s fun, rewarding, and can help others planning out their day or weekend!

## About
This software is distributed as a community submitted package as a part of [openhab-helper-libraries](https://github.com/openhab-scripters/openhab-helper-libraries). 

**weatherStationUploader** enables personal weather station owners to UPLOAD weather data in real time to Weather Underground.

#### Prerequisits
* [meteocalc](https://github.com/OH-Jython-Scripters/weatherStationUploader/blob/master/README.md#about#meteocalc%20Installation)

## meteocalc Installation
* The [meteocalc library](https://pypi.org/project/meteocalc/) is used for some calculations in the script. It must be installed and accessible from within the openHAB jsr223 environment. There might be other ways but the following was done on a ubuntu installation to achieve that. It might look different on your system.

* `sudo pip install meteocalc`
* Now, openHAB jsr223 jython must be configured to find your downloaded python libraries. There are several ways to do this. You can add a -Dpython.path=mypath1:mypath2 to the OH2 script `/etc/default/openhab2`. For example `-Dpython.path=/etc/openhab2/automation/lib/python:/usr/local/lib/python2.7/dist-packages` You can also modify the sys.path list in a Jython script that loads early (like a component script). Or as an alternative you can create a symlink like `sudo ln -s /usr/local/lib/python2.7/dist-packages/meteocalc /etc/openhab2/automation/lib/python/meteocalc` (In this example, the directory /etc/openhab2/automation/lib/python is already defined as openHAB's python library search path)
* Edit classutils.py, change line 5 to: `PYTHON2 = 2#sys.version_info.major`

## Installation
After the prerequisits are met:
* Register a new PWS (Personal Weather Station) at [Weather Underground](https://www.wunderground.com/personal-weather-station/signup). Note your station ID and station password.

* Follow the instruction for how to [install a community submitted package](../../../Docs/Python/Community.md#installation).

* Edit the configuration dictionary `weatherStationUploader_configuration` that you've previously inserted into the openhab-helper-libraries configuration file. Make sure that the Weather Underground station ID and password are set to match the credentials that you were given when registering your PWS.

* Reload the openhab-helper-libraries configuration as described in [Reload configuration](../../../Docs/But-How-Do-I.md).

* Watch the openHAB debug output for any errors or warnings.

## Disclaimer
THIS SOFTWARE IS PROVIDED BY THE AUTHOR "AS IS" AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE DISCLAIMED. IN NO EVENT SHALL THE AUTHOR BE LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
