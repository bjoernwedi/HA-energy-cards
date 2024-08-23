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

### building the template sensors
While we had prepared the helper templates, it becomes now time to create template sensors for our values we want to maintain in the coards.
All my different sensor follow the same style and just new names/referneces:
```
template:
  - sensor:
      - unique_id: "power_state_now"              <= to be able to manage them in UI, you shall give them unique ids
        unit_of_measurement: kW                   <= units simplify classifications
        device_class: power                       <= categories help either
        state_class: measurement                  <= and we have measurements here
```
So far nothing special, but now comes the tricky part.
We want to have friendly name to become dynamic, icon to change according to state, availability to change, and color being availacle.
Therefore remember the jinja templates from ontop. we always define the attribute with template, import the macro and make use of it:

For map_w_to_state or map_w_to_kw:
```
        name: >
            {% from 'energy_templates.jinja' import map_w_to_state %}{{ map_w_to_state('sensor.sonnen_original_messwerte_statusapi','GridFeedIn_W','Einspeisung','Netzbezug','Selbstversorgung') }}
        icon: >
            {% from 'energy_templates.jinja' import map_w_to_state %}{{ map_w_to_state('sensor.sonnen_original_messwerte_statusapi','GridFeedIn_W','mdi:home-export-outline','mdi:home-import-outline','mdi:home-lightning-bolt-outline') }}
        state: > 
            {% from 'energy_templates.jinja' import map_w_to_kw %}{{ map_w_to_kw('sensor.sonnen_original_messwerte_statusapi','GridFeedIn_W') }}
```
Or even other conversion ( ** note olor being one of the state attributes:** home assistant has default device category icn colors with available and unavailable colors, but we want to have positive/neutral/negative, so we maintain the state attribute we have to reference to in card_mod card)
```
        availability: >
            {{ iif(has_value('sensor.sonnen_original_messwerte_statusapi'),'on','off') }}
        attributes: 
            color: >
                {% from 'energy_templates.jinja' import map_w_to_state %}{{ map_w_to_state('sensor.sonnen_original_messwerte_statusapi','GridFeedIn_W','#8BC34A','#FF5722','var(--state-unavailable-color)') }}
            watts: >
                {{ state_attr('sensor.sonnen_original_messwerte_statusapi','GridFeedIn_W') | int(0) }}
            label: "power_state_now"
```
