# Home Assistant Door Control Blueprints

**DISCLAIMER:**  
These blueprints are provided "as is" without any warranty, express or implied. The creator of these blueprints makes no representations or warranties regarding the accuracy, reliability, or suitability of these automations for any purpose. Use these blueprints at your own risk. Under no circumstances shall the creator be held liable for any damages, losses, or adverse consequences arising from the implementation or use of these blueprints—including, without limitation, unintended door unlocks or locks. It is the responsibility of the user to test these blueprints thoroughly in a controlled environment before deploying them in production. By using these blueprints, you acknowledge that you are solely responsible for any modifications necessary to ensure safe and appropriate operation.

## Requirements

- **hass-unifi-access**  
  Install the [hass-unifi-access](https://github.com/imhotep/hass-unifi-access) integration in your Home Assistant instance. A big thanks to its developer!
- **Calendar Integration**  
  At least one calendar must be set up in Home Assistant for these blueprints to function.
- **Input Helpers & Select Entities**  
  The blueprints assume you have or will create:
  - An **input_text** helper for storing the door keyword (e.g., `*` by default).
  - A **select** entity to control the door lock mode (e.g., options like `keep_unlock` to unlock and `reset` to lock).
- **Mobile Device Notifications**  
  These blueprints use a device selector (mobile_app integration) to send notifications to your mobile device.

## Blueprints Overview

### 1. Unlock Door Before Event with Input Text Keyword and Notification Device

**Purpose:**  
Unlocks a door a specified duration before an event begins if the event’s title contains the door keyword (stored in an input_text helper). The automation checks that the door is currently locked before unlocking it and sends a notification to a selected device.

**Key Inputs:**
- **Calendar Entity:** Select which calendar to monitor.
- **Door Keyword Input Text:** Select the input_text helper that holds the door keyword (default `*`).
- **Unlock Offset:** Set the duration (e.g., `-00:30:00`) before the event start when the door should unlock.
- **Door Lock Entity & Rule Select:** Specifies the door lock to check and the select entity to change the lock rule.
- **Notification Device:** Select the mobile device to receive notifications.

---

### 2. Lock Door on Canceled or No Active Event with Full Keyword Check

**Purpose:**  
Periodically locks the door if either:
- An active event is present whose summary contains both the cancellation keyword (default `"canceled"`) and the door keyword (from the input_text helper) as full words, or  
- No active event is detected at all.

**Key Inputs:**
- **Calendar Entity:** Select which calendar to monitor.
- **Cancellation Keyword:** Text input for the cancellation indicator (default `"canceled"`).
- **Door Keyword Input Text:** Select the input_text helper for the door keyword (default `*`).
- **Door Lock Entity & Rule Select:** Specifies the door lock and the select entity for controlling the lock mode (setting to `"reset"` locks the door).
- **Notification Device:** The mobile device to send notifications to.

**How It Works:**  
This blueprint runs every minute. It first checks that the door is unlocked. Then, using a template, it determines whether there is an active event (based on start and end times) and whether the event summary contains both keywords as full words. If either a canceled event is detected or no active event exists, it locks the door and sends a notification.

---

### 3. Nightly Door Safeguard On/Off

**Purpose:**  
Temporarily disables the unlock automation during a defined nighttime period and locks the door, then re-enables the unlock automation in the morning.

**Key Inputs:**
- **Unlock Automation:** The automation that unlocks the door (to be disabled at night and re-enabled in the morning).
- **Disable Time & Enable Time:** Times at which the unlock automation is turned off and back on, respectively (default disable at `22:00:00` and enable at `05:00:00`).
- **Door Lock Rule Select:** The select entity that controls the door lock mode (setting to `"reset"` locks the door).
- **Notification Device:** The mobile device to send notifications to.

**How It Works:**  
At the disable time, the blueprint turns off the unlock automation and immediately locks the door by setting the door lock rule to `"reset"`, sending a notification. At the enable time, it turns the unlock automation back on and sends a notification.

---

### 4. Periodic Door Unlock Fail-Safe (Unlock Only) with Duration Frequency

**Purpose:**  
Periodically checks if there is an active event that contains the door keyword (and does not include the cancellation keyword) and, if the door is locked, unlocks it. This fail-safe ensures the door remains unlocked during events even if the event-based trigger is missed.

**Key Inputs:**
- **Check Frequency:** A duration input (HH:MM:SS) specifying how often the automation should check (e.g., `00:01:00` for every minute).
- **Calendar Entity:** The calendar to monitor.
- **Door Keyword Input Text:** Select the input_text helper holding the door keyword.
- **Cancellation Keyword:** Text input for the cancellation indicator (default `"canceled"`).
- **Door Lock Entity & Rule Select:** Specifies the door lock (the automation acts only if locked) and the select entity to change the lock mode (setting to `"keep_unlock"` unlocks the door).
- **Notification Device:** The mobile device for notifications.

**How It Works:**  
The blueprint triggers periodically at the defined frequency. It checks that the door is locked and then verifies if an active event is present by comparing the current time to the event’s start and end times (from the calendar’s attributes). It also confirms that the event summary includes the door keyword (as a full word) and does not include the cancellation keyword. If these conditions are met, it unlocks the door by setting the door lock rule to `"keep_unlock"` and sends a notification.

---

## Usage Notes

- **Installation:**  
  Before using these blueprints, install the [hass-unifi-access](https://github.com/imhotep/hass-unifi-access) integration and configure it as required.
  
- **Calendar Requirement:**  
  At least one calendar must be set up in Home Assistant since the automations rely on calendar events for door control actions.
  
- **Customization:**  
  You can customize keywords, time offsets, check frequency, and notification settings via the blueprint inputs.
  
- **Testing:**  
  It is recommended to thoroughly test these blueprints in a controlled environment before deploying them to ensure they operate correctly with your door lock configuration.

---

Happy automating!
