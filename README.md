# HA-energy-cards
my way to clean graph cards without specific development based on default custom cards

## Where it started
While I was applying photovoltaik to my house and started also with a battery for the nights, I had home assistant already installed for many devices, but also smart gardening and irrigation.
Because i then started using an EV and applied a wallbox at home, the energy chapter in one place became a topic.
Unsatisfyingly home assistant did not give me the apperance I liked to have to combine solar panel production, battery management, wallbox, car. at least not out of the box.
Due to the need of some reconstruction in my power distribution, I had the opportunity to als apply some energy meters per floor.

Finally the idea for a new dashboard with default layout tiles was born to bring all these different digital inputs together.

A card having the visual representation of the meter, the days measurement, the last days one, the actual value, a specific dynamic state, icon and color.

### getting the sensors ready
Before getting started visually, I tried to get the sensor templates ready to maintain all the values, but unfortunatly it went complicated immediate.
You can find them under [/custom_templates/energy_templates.jinja](./custom_templates/energy_templates.jinja):
#### macro map_w_to_kw(sensor, attribute, precision=int(1))
Is meant to apply a conversion from W to kW for a specific sensors state attribute, with a given precision you like, defaulted to 1.
but inside you will also find a reference to an manual input helper to reduce blur/noise to have a clean "0" (inactive) state from -/+ input value.
```
{%  
    macro map_w_to_kw(sensor, attribute, precision=int(1)) %}{% 
        set raw = state_attr(sensor,attribute)|float(0) %}{% 
        set min = states('input_number.treshold_w')|float(100) %}{%                     <= the treshold to supress noise around 0
        if(raw|abs>min) %}{{ ( ( raw / 1000) | round( precision | int( 1 ) ) ) }}{%     <= the comparism before converting
            else %}{{ float(0) }}{% 
        endif %}{%  
    endmacro 
%}
```
#### macro map_w_to_state(sensor, attribute, positive, negative='', neutral='')
Is almost the same as above, but converts a state attribute of an entity into positive, negative and normalized neutral values/strings to map negative, positive and neutral to state labels.
#### macro map_w_state(sensor, positive, negative='', neutral='')
Is the same, but is made for an entity state and not for an entity's attribute state.
#### macro map_w_to_availability(sensor, attribute, precision=int(0))Based on the before values just turns into on and off
