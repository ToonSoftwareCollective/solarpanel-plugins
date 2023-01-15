# Home Assistant sensor

De home assistant plugin leest een sensor met de naam 'solar_panel' uit. Deze sensor dien je zelf te definieren in een sensor template. Zie hieronder voor een voorbeeld. 

### Sensor template
    - sensor:
        - name: "Solar panel"
          unique_id: 36164133-d6cb-4d56-9c8b-38c634fc2c5f
          state: >
            {% if states.sensor.growatt_total_energy_lifetime is defined and states.sensor.growatt_total_energy_lifetime.state not in ['unavailable', 'none', 'unknown', '0'] %}
                {{ (states.sensor.growatt_total_energy_lifetime.state | float(0)) * 1000 }}
            {% else %}
                {{ states.input_number.solar_panels_total_lifetime_wh.state | float }}
            {% endif %}
          icon: >
            mdi:solar-power
          attributes:
            current: >
              {% if states.sensor.growatt_total_power is defined %}
                  {{ states.sensor.growatt_total_power.state | float(0) }}
              {% else %}
                  'unavailable'
              {% endif %}
            today: >
              {% if states.sensor.growatt_total_energy_today is defined and states.sensor.growatt_total_energy_today.state not in ['unavailable', 'none', 'unknown', '0'] %}
                  {{ ( states.sensor.growatt_total_energy_today.state | float(0)) * 1000 }}
              {% elif states.sensor.growatt_total_energy_today is defined and states.sensor.growatt_total_energy_today.state in ['unavailable', 'none', 'unknown', '0'] %}
                  {{ states.input_number.solar_panels_today_wh.state | float }}
              {% else %}
                  'unavailable'
              {% endif %}
            total: >
              {{ states.sensor.solar_panel.state }}


### Automation
Ik maak gebruik van een ESPHome dongle in mijn growatt inverter die de data direct naar mijn home assistant server stuurt, maar deze dongle raakt 'unavailale' wanneer de inverter uitstaat (wanneer de zon niet (meer) schijnt). Hierdoor zou de data die Toon ophaalt regelmatig leeg kunnen zijn. Daarom maak ik gebruik van 2 input_number helpers: 'input_number.solar_panels_total_lifetime_wh' en 'input_number.solar_panels_today_wh'. Deze worden door een automation gevuld.

    alias: "Systeem: Solar panels"
    description: ""
    trigger:
      - platform: state
        entity_id:
          - sensor.growatt_total_energy_lifetime
        id: lifetime
      - platform: state
        entity_id:
          - sensor.growatt_total_energy_today
        id: today
    condition:
      - condition: or
        conditions:
          - condition: and
            conditions:
              - condition: not
                conditions:
                  - condition: state
                    entity_id: sensor.growatt_total_energy_lifetime
                    state: unavailable
              - condition: not
                conditions:
                  - condition: state
                    entity_id: sensor.growatt_total_energy_lifetime
                    state: unknown
          - condition: and
            conditions:
              - condition: not
                conditions:
                  - condition: state
                    entity_id: sensor.growatt_total_energy_today
                    state: unavailable
              - condition: not
                conditions:
                  - condition: state
                    entity_id: sensor.growatt_total_energy_today
                    state: unknown
    action:
      - if:
          - condition: trigger
            id: lifetime
        then:
          - service: input_number.set_value
            data:
              value: >-
                {{ (states.sensor.growatt_total_energy_lifetime.state | float(0)) * 1000 }}
            target:
              entity_id: input_number.solar_panels_total_lifetime_wh
      - if:
          - condition: trigger
            id: today
        then:
          - service: input_number.set_value
            data:
              value: >-
                {{ ( states.sensor.growatt_total_energy_today.state | float(0)) * 1000 }}
            target:
              entity_id: input_number.solar_panels_today_wh
    mode: single


