# Furnace filter runtime sensor (0.96 and newer)

This project uses the variable custom component, template sensors, a script, history stats sensors, and an automation to track total HVAC runtime since the furnace filter was last changed. 

- The `furnace_filter` variable value is a time/date string.
- The `hvac_runtime` variable value is the total runtime since the filter was last changed.
- The script sets the `furnace_filter` value to the current date/time and resets the `hvac_runtime` variable. You can call this script in an entity button in your frontend to conveniently reset everything after you change the filter.
- The history stats sensors record the amount of time (in hours) that the system has been heating/cooling starting at midnight on the day the filter was changed and ending at the time the filter was changed. This is used to not double-count runtime when the filter is marked as changed and the `hvac_runtime` variable is updated. Some are also used to calculate a rolling 7-day average to determine approximately how many days are remaining until the next filter change will be required. It assumes the filter needs to be changed after 200 hours of use, but this can of course be changed.
- The `hvac_action` template sensor is required for HASS versions 0.96 and newer to allow the history stats sensors to track heating/cooling runtime due to changes in the Climate component.
- A template sensor adds the value in `hvac_runtime` and the heating/cooling runtime for the current day, and an automation runs each night to update `hvac_runtime` with this value so that the total runtime is permanently stored.

## Notable features and limitations
- Only requires 1 day of history (or 7 days if you want to calculate and display an estimate of when the filter will need to be changed based on the rolling 7-day average)
- The automation to store the total runtime each night runs at 23:59:30 because if I changed it to 23:59:59 I'm scared the system might be busy and the automation won't fire on time which would lead to a substantial loss of runtime. So with it set to 23:59:30 there will be up to 30 seconds of runtime not stored each day. You can of course change the time trigger to 23:59:59 if you're less paranoid than me which only leads to up to 1 sec of drift per day. Either way, it's good enough for government work.

Thanks to rogro82 for his [variable custom component](https://github.com/rogro82/hass-variables).
