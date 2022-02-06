---
title: "Oracle Apex Subscription Button"
subtitle: How to create a Subscription or Like Button
categories:
  - Blog
tags:
  - apex
  - JavaScript
  - PL/SQL
---

### Can we have a like or subscribe Button on an Oracle Apex Page? Yes, we can...

![subscription button](..\assets\images\subscription.png)

------

This solution consists of four  parts. One Table, a button, an application or page process and some JavaScript to do the magic.

For this example I created this table where every subscribed event from a user is stored.

```sql
create table "SA_SUBSCRIPTION" (
   "APP_USER" varchar2(256 byte)
 , "EVENT" varchar2(16 byte)
);

comment on column "SA_SUBSCRIPTION"."APP_USER" is 'Application User';
comment on column "SA_SUBSCRIPTION"."EVENT" is 'Subscribed event';
```



In the next step we create on or more buttons with the following settings 

| Appearance              | Value           |
| ----------------------- | --------------- |
| Button Template         | Icon            |
| Template Options, Style | Display as Link |
| Icon                    | fa-bell         |

The only important thing is the Custom Attribute. Set the data-event attribute to something meaningful. Every different event has it's own!

| Advanced          | Value                       |
| ----------------- | --------------------------- |
| Custom Attributes | data-event="**EVENT-MAIL**" |

After that we have to write an application process if we need this function on multiple pages or a page process. This process is of type Ajax-Callback. The button color and title could be set to your needs.

```sql
declare
  type T_EVENTS_TAB is
    table of SA_SUBSCRIPTION.EVENT%type;
  L_EVENTS    T_EVENTS_TAB;
  L_EVENT     SA_SUBSCRIPTION.EVENT%type;
  K_ACTIVE constant boolean := true;
  K_INACTIVE  constant boolean := false;
begin

  -- Open JSON Object
  APEX_JSON.OPEN_OBJECT;

  -- New Object for Button Settings. Adapt to your needs
  APEX_JSON.OPEN_OBJECT('settings');
    APEX_JSON.write('active_text', 'Unsubscribe from this event');
    APEX_JSON.write('inactive_text', 'Subscribe to this event');
    APEX_JSON.write('active_color', '#f50537');
    APEX_JSON.write('inactive_color', '#000000');
  APEX_JSON.CLOSE_OBJECT;

  -- New Object for the events
  APEX_JSON.OPEN_OBJECT('events');

  -- Process Button click
  if APEX_APPLICATION.G_X01 is not null then
    -- Button was clicked
      delete SA_SUBSCRIPTION
       where APP_USER = :APP_USER
         and EVENT = APEX_APPLICATION.G_X01;

      if sql%ROWCOUNT = 0 then
        -- No reocrd was found. Insert new record
        insert into SA_SUBSCRIPTION (APP_USER, EVENT) 
          values (:APP_USER, APEX_APPLICATION.G_X01);
        APEX_JSON.write(APEX_APPLICATION.G_X01, K_ACTIVE);
      else
        -- Record was deleted 
        APEX_JSON.write(APEX_APPLICATION.G_X01, K_INACTIVE);
      end if;
  
  end if;
  
  -- Process for all Buttons on Page Load
  if APEX_APPLICATION.G_F01.count > 0 then

    -- Load user settings for events into new array. Doesn't take care of events from APEX_APPLICATION.G_F01 :(
    select EVENT
      bulk collect
      into L_EVENTS
      from SA_SUBSCRIPTION
     where APP_USER = :APP_USER;

    -- Loop input array and check against new array
    for i in 1..APEX_APPLICATION.G_F01.count loop
    
      -- Active because element of new array
      if APEX_APPLICATION.G_F01(i) member of L_EVENTS then
        APEX_JSON.write(APEX_APPLICATION.G_F01(i), K_ACTIVE);
      
      -- Inactive because not an element of array
      else
        APEX_JSON.write(APEX_APPLICATION.G_F01(i), K_INACTIVE);
      end if;
      
    end loop;

  end if;

  APEX_JSON.CLOSE_ALL;
  
exception 
  when others then
    APEX_JSON.INITIALIZE_OUTPUT;
    APEX_JSON.OPEN_OBJECT;
    if APEX_APPLICATION.g_debug then
      APEX_JSON.WRITE('error', 'Error while processing subscription.<br><i>'   || sqlerrm || '</i>');
    else
      APEX_JSON.WRITE('error', 'Error while processing subscription');
    end if;
    APEX_JSON.CLOSE_OBJECT;    
end;
```

With the process in place we need a JavaScript function to call the process und set the button state. For this I putted everything in a file and uploaded this to the static files. Then load this file on every page u need it or gloabl.

```javascript
/**
 * @function Subscripe to a channel on button click and show status on button with custom color
 * @param {*} pX01 
 */
function subscription(pX01) {
    //create an array and store data-event
    var dataArray = [];

    // Collect events only if X01 is not defined
    if (pX01 === undefined) {
        apex.jQuery('button[data-event]').each(function () {
            dataArray.push(apex.jQuery(this).attr('data-event'));
        });
    }

    console.log(pX01);

    // Call process
    apex.server.process('CHECK_EVENT', {
        f01: dataArray,
        x01: pX01
    }, {
        success: function (data) {
            // loop json object and set class
            for (const [key, value] of Object.entries(data.events)) {
                if (value === true) {
                    apex.jQuery('[data-event="' + key + '"] span').css('color', data.settings.active_color);
                    apex.jQuery('[data-event="' + key + '"]').prop('title', data.settings.active_text);
                } else {
                    apex.jQuery('[data-event="' + key + '"] span').css('color', data.settings.inactive_color);
                    apex.jQuery('[data-event="' + key + '"]').prop('title', data.settings.inactive_text);
                }

            }
        }
    });
}

(function init() {
    // Init subscription after page load
    apex.jQuery(window).on('theme42ready', function () {
        // Run after ui elements rendered on page
        //console.log('Init subscription on page.');
        subscription();
    })

    // Init onClick event for buttons with attribute data-event
    apex.jQuery('button[data-event]').on('click', function () {
        //console.log(apex.jQuery(this).attr('data-event'));
        subscription(apex.jQuery(this).attr('data-event'));
    })
})();
```
