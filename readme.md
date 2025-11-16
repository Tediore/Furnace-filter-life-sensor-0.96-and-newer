## NOTE: This is no longer being maintained. 
### I realized that it was way overengineered, so I moved to a much simpler setup which you can see below. Naturally, you'll need to substitute your own notification service and thermostat entity (like with the original package)

```yaml
timer:
  furnace_filter:
    name: Furnace filter life remaining
    duration: '300:00:00'
    icon: mdi:air-filter
    restore: True

template:
- button:
  - name: Filter replaced
    unique_id: 'replace-furnace-filter-btn'
    default_entity_id: button.furnace_filter_replaced
    press:
      service: timer.finish
      entity_id: timer.furnace_filter

- sensor:
  - name: Furnace filter life remaining
    unique_id: 'furnace-filter-life-rmng'
    default_entity_id: sensor.furnace_filter_life
    state: >
      {% if state_attr('timer.furnace_filter', 'remaining') == None %}
        {{ state_attr('timer.furnace_filter', 'duration').split(':')[0] }}
      {% else %}
        {% set timer_list = state_attr('timer.furnace_filter', 'remaining').split(":") %}
        {{ timer_list[0] | int(0) + ((timer_list[1] | int(0) / 60) | round(1)) }}
      {% endif %}
    icon: 'mdi:air-filter'
    unit_of_measurement: 'h'

  - name: HVAC action
    unique_id: 'hvac-action-sensor'
    default_entity_id: sensor.hvac_action
    state: "{{ state_attr('climate.thermostat', 'hvac_action') }}"

automation:
- alias: Start/stop timer with HVAC
  id: 'start-stop-timer-hvac'
  initial_state: true
  trigger:
  - platform: state
    id: 'timer.start'
    entity_id: sensor.hvac_action
    to:
    - 'cooling'
    - 'heating'
    - 'fan'
  - platform: state
    id: 'timer.pause'
    entity_id: sensor.hvac_action
    to: 'idle'
  action:
  - service: "{{ trigger.id }}"
    entity_id: timer.furnace_filter

- alias: Furnace filter replacement notification
  id: 'furnace-filter-replacement-notif'
  initial_state: true
  trigger:
  - platform: event
    event_type: timer.finished
    event_data:
      entity_id: timer.furnace_filter
  action:
  - service: notify.gotify_high
    data:
      title: Replace furnace filter
      message: Filter has been used for 300 hours.
```

## Furnace filter runtime sensor (Home Assistant >= 0.96)

*This is a re-release of the previous filter runtime sensor in `package` format instead of in separate pieces. Not sure why I didn't do it this way in the first place.*

This package uses the input_text and input_number integrations, template sensors, a script, history stats sensors, and an automation to track and display the total HVAC runtime since the furnace filter was last changed. It also calculates an estimate of the number of days until your filter will need to be changed. [History stats](https://www.home-assistant.io/integrations/history_stats/) are great, but they're limited to what you have `purge_keep_days` set to in `recorder`. This package only requires one day of history to record runtime and seven days to calculate remaining runtime based on a rolling seven-day average.

- The `furnace_filter` input_text is a time/date string and is used to record when the filter was last changed.
- The `hvac_runtime` input_number stores the total runtime since the filter was last changed.
- The `replacement_threshold` input_number is used to set the number of hours that you expect the filter to last. I like sliders, but you can of course change the configuration to box if you prefer to type out a number.
- The script sets `furnace_filter` to the current date/time and resets `hvac_runtime`. You can call this script in an entity button in your frontend to conveniently reset everything after you change the filter.
- The history stats sensors record the amount of time (in hours) that the system has been heating/cooling starting at midnight on the day the filter was changed and ending at the time the filter was changed. This is used to not double-count runtime when the filter is marked as changed and `hvac_runtime` is updated. Some are also used to calculate a rolling seven-day average to determine approximately how many days are remaining until the next filter change will be required. The value that `replacement_threshold` is set to is used in this calculation.
- The `hvac_action` template sensor is required for HASS versions 0.96 and newer to allow the history stats sensors to track heating/cooling runtime due to changes in the Climate component. **NOTE: You'll need to change the entity ID in this sensor to match your thermostat entity ID, otherwise this won't work.**
- A template sensor adds the value in `hvac_runtime` and the heating/cooling runtime for the current day, and an automation runs each night to update `hvac_runtime` with this value so that the total runtime is permanently stored.


### Prerequisites
- You must have a thermostat connected to Home Assistant via one of the Climate platforms.
- You must be running Home Assistant version 0.96 or newer (otherwise you'll need to tweak this a little bit).
- `packages` [must be defined in your configuration.yaml](https://www.home-assistant.io/docs/configuration/packages/). Don't be scared, it's not difficult.


### FAQ
Q. Why did you use an `input_text` instead of `input_datetime` for the furnace filter change date?<br>
A. I have no idea.

Q. What? But that doesn't make any sense.<br>
A. I know, but it works.

Q. But I don't like using `packages`.<br>
A. That's not a question. And too bad.

Q. Are you really as good looking in person as you are in your avatar?<br>
A. Even more, actually.
