# signalk-sensor-log

Log and chart sensor data using round-robin databases.

At a regular, user-defined, sampling _interval_ __signalk-sensor-log__
retrieves a set of user selected sensor _values_ from the host Signal K Node
server and saves each of these values into one or more user defined
round-robin _databases_.

Values in each database are consolidated over user defined time periods in a
way which prevents database expansion at the expense of some loss of detail at
larger timescales.
By default, the plugin consolodates data over hour, day, week, month and
year timescales.

Data derived from each channel can be rendered as part of one or more chart
collections (a chart collection is just a set of charts which depict the same
data values over each of the defined timescales).
The plugin includes a simple web application which can be used to display any
generated charts.

Database update and rendering services are implemented as stand-alone daemons
which run as separate processes from the host Signal K Node Server and have
minimal impact upon server operation even when many sensor channels are being
logged.

The plugin offers many user configuration options, but is designed to work
'out of the box' in a way that may suit many users.

Some examples of the type of data logging that can be undertaken are
illustrated below.

__Example 1__  
Power levels reported from the host vessel's inverter/charger show changes in
electrical power consumption and use.

![Webapp screenshot](readme/acpower.png)

__Example 2__
Readings from tank level sensors are plotted to illustrate changes in tank
content over time.

![Webapp screenshot](readme/tanks.png)

__Example 3__
NMEA switchbank state changes over time.

#include readme/system-requirements.md

#include readme/installation.md

#include readme/use.md

## Usage

### Running the plugin for the first time

In the server main menu navigate to _Server->Plugin config_ and select _Sensor
Log_to access the plugin configuration screen.

The configuration screen consist of a number of collapsible sections, each
concerned with some aspect of the plugin's operation.
Proceed with first configuration by:

1. Expand the _rrdservices_ configuration tab and confirm that the service
   connection settings conform to the connection
1. Expand the _rrdserver_ configuration tab and confirm that the server
   connection details correspond with the service configuration you
   implemented in the previous section.

   ![rrdserver configuration tab](readme/rrdserver-tab.png)

   Make any necessary changes and close the tab.

2. Expand the _sensor_ tab.
   
   ![rrdserver sensor tab](readme/sensor-tab.png)

   __signalk-sensor-log__ defaults to attempting to recover tank level and
   electrical power data streams from the Signal K Node server.
   If your system has this data available, then you will see a list of the
   detected electrical and tank sensors; if this data is not available on
   your system or you are interested not in it, but in something else, then
   you need to modify the value of the option _Regex to select sensor paths_.

   Any
   [regular exression]()
   supplied in this field will be used to filter the list of available sensor
   paths returned by the Signal K Node server into the list that will form the
   basis of __signalk-sensor-log__'s processing. 
   You can review the server's comprehensive list by pointing your browser at
   `http://_your_server_address_:_your_server_port_/signalk/v1/api/vessels/self/`.

   The selection you make here only needs to be approximate: individual sensor
   paths can be pruned at a later stage.
   If you wish, you can simply enter ".*" as the regular expression and grab
   all of the available sensor data.


When opened for the first time the configuration page will be populated with
an entry for every sensor registered on the host Signal K server.
Most likely you will not want to monitor all of these: indeed some will most
likely carry no data and the chances are that you will not know which these
are!

A reasonable strategy immediately after installation is to allow the plugin to
run with its default configuration for a while and then review and probably
prune the list of monitored sensors before re-initialising the plugin database
so that it only logs the data you are interested in.

To achieve this, check the _Active_ option (at the top of the configuration
form) and the option _Report power levels to console_ before clicking the
_Submit_ button (at the bottom of the configuration form).

The plugin should now be listed on the Signal K Node server _Dashboard_ where
a status message will indicate that stream data is being processed.

After ten minutes or so, navigate to _Webapps_ and open the
_Signalk-Power-Monitor_ application in order to review harvested data and make
some choices about what you do and do not need to keep track of.

### Configuring the plugin data inputs

Once you are confident about the data streams you need to process, return to
the plugin configuration page and delete any redundant entries from the list
of _Power sensors_.

It is a good idea at this stage to give each of the remaining sensors a
meaningful name that can be used in graphical display.

If you have deleted any redundant data streams, then you may wish to check
the _Recreate database_ option and so recover a little bit of disk space.

### Configuring graphical output

By default the plugin generates a set of five charts which use a simple line
graph to illustrate power use for all of the monitored sensor streams over the
past hour, day, week, month and year.

My installation is based around a Victron Quattro inverter/charger and on
shore-power the device reports three sensor streams: power output to the
battery charger, power output to the inverter and power input to the system.
The default chart looks like this.

![Webapp screenshot](readme/screenshot.png)

In some ways, the power input value is just clutter and a chart which displayed
the two output sensor values stacked on top of one another is a simpler and,
arguably, more meaningful way of rendering this data.
This type of chart can be achieved by checking the _Stack_ option on the two
output sensor channels.
The resulting chart looks like this.

![Webapp screenshot](readme/screenshot.png)

Be aware that the when a chart is generated sensor streams are plotted in the
order in which they appear in the sensor list: you may want to re-order the
list of sensors so that the chart displays things in the most attractive way.

Otherwise, continue by reviewing the screen options (all of which are
described below) making changes which match your requirement.
When you are satisfied, check click _Submit_.

### Plugin configuration

#### systemd/service.conf

__signalk-sensor-log__ interacts with two stand-alone service daemons and
inevitably there is a handful of configuration settings which must be shared.
These shared settings are held in the file `systemd/service.conf` which is
read by both __systemd__ (as it starts __rrdcached__ and __rrdchartd__) and
the plugin.

Settings in `systemd/service.conf` are fully documented through comments in the
file itself and these values must be reviewed and changed if necessary as part
of plugin commissioning.

#### Configuration through Signal K Node 



#### General configuration options

__Database pathname__  
Specifies the pathname used for the plugin's round-robin database.
The default behaviour is to create a file called `database.rrd` in the plugin
install directory.

If you change this value the plugin will immediately begin using the specified
database, automatically creating a new one if the named database does not
exist.
Note that if you specify a pathname outside the plugin install directory, then
you must make sure that the directory path exists and has appropriate access
permissions to allow the database to be created and updated by the owner of
the executing Signal K Node server.

__Database update interval__  
Species the frequency in seconds at which data should be sampled from the host
system and written to the database.
The default value is 10 seconds and this is probably close to the highest
practical update frequency on most systems.

If you change this value after initial configuration the plugin will delete
the existing database and create a new one.

__Database advanced options__  
If checked, these options are applied once when the _Submit_ button is clicked.

    __Regenerate database__  
    Deletes the existing database and creates a new one using the current
    option settings.
    
    __Re-initialise database__  
    Deletes the existing database, re-builds the plugin's _Sensor list_ from
    the Signal K Node server's list of available channels and creates a new 
    database capable of holding the whole sensor set.
    No option settings other than the _Sensor list_ are changed.
    
__Chart directory__  
Specifies the directory where the plugin will create its charts.
The default behaviour is to store charts in the
`_plugin_install_directory_/public/` folder.

You can change this location, but the new location must have permissions which
will allow files to be created and updated by the owner of the executing Signal
K Node server.
Also note that the plugin webapp (see below) expects its charts to be available
in the default location.

Setting this option to the empty string will disable chart generation.

__Report database updates to console__  
Specifies whether or not the plugin should issue a terse summary of the data
values being written to the database each time an update occurs.
The default behaviour is not to report in this way.

### Sensors

![Sensors](readme/sensorconfig.png)

The _Sensors_ configuration consists of a list of sensors identified by the
plugin.
Each sensor in the list is configured by the following set of options.

__Sensor path__  
Shows the Signal K data path identifying this sensor.
The value shown is generated by the server and cannot be changed.

__Sensor name__  
Specifies the name to use for this sensor in all display contexts.
The default value is a probably unique, possibly inscrutable, token derived
from the sensor path.
You will probably want to change this into something more meaningful.

__Color to use when rendering this sensor__  
Specifies the color to be used when rendering this sensor's data on a chart.
The default value is sequentially selected from a palette of twenty easily
differentiated colors.

__Display groups which include this sensor__  
Specifies a space-separated list of display groups which include this sensor.
By default, all sensors are assigned to the display group ALL.
If you enter the name of a non-existant display greoup, then the plugin will
automatically create a new display group (this will not appear in the plugin
configuration screen until _Submit_ has been clicked to commit the change).

__Multiplier for sensor values__  
Specifies a multiplication factor which should be applied to this sensor's
data values.
The default value is 1 (i.e. sensor values are unchanged).
If you prefer values which are reported as ratios to be handled as percentages
then set the sensor multiplier to 100.

__Sensor options --> Stackable__  
Specifies whether or not the data from this sensor could reasonably be
stacked with other values in a chart.
The default value is true.
Note that setting this option only specifies whether or not the associated
data can be stacked, a _Display group_ option (see below) will determine
whether or not the sensor data is rendered in this way on an output chart.
You can uncheck this option  to ensure that a sensor value which is
inherently discrete cannot be rendered in a misleading way.

### Display groups

![Display groups](readme/displaygroup.png)

This _Display groups_ configuration consists of a list of display group entries
creates dynamically from the display group names supplied in the _Sensors_
configuration.
Each display group in the list is configured by the following options.

__Display group id__  
Specifies a unique identifier for this display group.
This value is created by the plugin from a display group identifier specified
in the _Sensors_ list and cannot be changed.

__Chart title__  
Specifies a title to be used for this display group's webpage and in each of
the generated charts.
Default value is something innocuous.
You will probably want to change this.

__Chart y-axis label__  
Specifies the text to be used as a label on the vertical axis of charts in this
display group.
Default value is something innocuous.
You will probably want to change this to something that describes at least the
units of the displayed data.

__Maximum y-axis value__  
Specifies the maximum value that can be displayed on the the y-axis.
The default value of 0 (zero) indicates that the axis should be automatically
scaled in response to the maximum value in the displayed data.

__Chart options --> Stack graph data__  
When checked, selects a stacked-area chart rather than a line graph.
Defaults to unchecked.
All data values included in the display group will be stacked except for those which
are marked in the _Sensors_ configuration as not stackable.

## Notifications, warnings and errors

## Version history

## License

__signalk-tank-monitor__ is released under Apache License 2.0.




__Example 3__  
available in the
Signal K Node server's electrical data set.
Electrical power reeadings are harvested from the host Signal K Node server and
stored in a round-robin database (in my case, the monitored data originates
from a Victron Quattro 8000 and is injected into Signal K by the
[__signalk-venus-plugin__](https://www.npmjs.com/package/signalk-venus-plugin)).

Periodically data from the database is used to generate graphs which illustrate
the changes in power usage over time.
The plugin makes an attempt to derive kWh energy usage from the reported power
readings.

