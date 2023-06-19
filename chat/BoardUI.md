[ui](https://developers.miro.com/docs/ui).BoardUI

-   [openPanel](https://developers.miro.com/docs/ui_boardui#openpanel)
-   [closePanel](https://developers.miro.com/docs/ui_boardui#closepanel)
-   [openModal](https://developers.miro.com/docs/ui_boardui#openmodal)
-   [closeModal](https://developers.miro.com/docs/ui_boardui#closemodal)

-   [on](https://developers.miro.com/docs/ui_boardui#on)
-   [off](https://developers.miro.com/docs/ui_boardui#off)

▸ **openPanel**(`options`): `Promise`<`void`\>

Opens a panel on the current board.  
The content displayed in the panel is fetched from the specified URL.

If you don't set a value for `height`, the panel height corresponds to the current viewport height.

-   Panel width: currently it's set to `368` [dp](https://en.wikipedia.org/wiki/Device-independent_pixel), fixed, padding included.  
    You should implement your app to adapt to content area width changes between `292` dp and `320` dp, and to accommodate a fixed panel width between `340` dp and `368` dp, padding included.
-   Left padding: `24` dp
-   Right padding: `24` dp

Example:

```
await miro.board.ui.openPanel({
  url: 'show-this-page-in-the-panel.html',
  height: 400,
});

// The panel is displayed on the board
```

The content that apps display on modals and panels opens inside iframes.  
iframes can request access to the following [permissions](https://developers.miro.com/reference/scopes#microphonelisten):

-   `microphone:listen`: access a user's microphone to record audio in an iframe.
-   `screen:record`: access a user's screen to record it in an iframe.
-   `webcam:record`: access a user's camera to record video.

See also:

-   [Add permission scopes to your app](https://developers.miro.com/docs/add-permission-scopes-to-your-app)

| Name | Type | Description |
| --- | --- | --- |
| `options` | `Object` | \- |
| `options.url` | `string` | Absolute or relative URL pointing to the content that you want to display in the panel.  
If you specify a relative URL, the URL path resolves relative to the app URL.  
The transport protocol must be HTTPS. |
| `options.height?` | `number` | Sets the height of the panel, in [dp](https://en.wikipedia.org/wiki/Device-independent_pixel).  
If you don't specify any height, the panel takes the height value of the current viewport. |

`Promise`<`void`\>

___

▸ **closePanel**(): `Promise`<`void`\>

Closes an open panel on the current board.

The method doesn't take any arguments.

ℹ️ Note:

-   To close an open panel, `closePanel` must execute after [`openPanel`](https://developers.miro.com/docs/ui_boardui#openpanel).
-   The app that closes the panel with `closePanel` must be the same that opened it with [`openPanel`](https://developers.miro.com/docs/ui_boardui#openpanel).

Example:

```
// Open a panel
await miro.board.ui.openPanel({
  url: 'show-this-page-in-the-panel.html',
  height: 400,
});

// Close the open panel
await miro.board.ui.closePanel();
```

`Promise`<`void`\>

___

▸ **openModal**(`options`): `Promise`<`void`\>

Opens a modal on the current board.  
The content displayed in the modal is fetched from the specified URL.

The max. width and height of the modal correspond to the width and height of the current viewport.  
The load timeout for the modal is 10 seconds.

Example:

```
await miro.board.ui.openModal({
  url: 'show-this-page-in-the-modal.html',
  width: 600,
  height: 400,
  fullscreen: false,
});

// The modal is displayed on the board
```

The content that apps display on modals and panels opens inside iframes.  
iframes can request access to the following [permissions](https://developers.miro.com/reference/scopes#microphonelisten):

-   `microphone:listen`: access a user's microphone to record audio in an iframe.
-   `screen:record`: access a user's screen to record it in an iframe.
-   `webcam:record`: access a user's camera to record video.

See also:

-   [Add permission scopes to your app](https://developers.miro.com/docs/add-permission-scopes-to-your-app)

| Name | Type | Description |
| --- | --- | --- |
| `options` | `Object` | \- |
| `options.url` | `string` | Absolute or relative URL pointing to the content that you want to display in the modal.  
If you specify a relative URL, the URL path resolves relative to the app URL.  
The transport protocol must be HTTPS. |
| `options.height?` | `number` | Sets the height of the modal, in [dp](https://en.wikipedia.org/wiki/Device-independent_pixel).  
The max. height of the modal corresponds to the height of the current viewport.  
Default: `600` |
| `options.width?` | `number` | Sets the width of the modal, in [dp](https://en.wikipedia.org/wiki/Device-independent_pixel).  
The max. width of the modal corresponds to the width of the current viewport.  
Default: `800` |
| `options.fullscreen?` | `boolean` | Set it to:  
-   `false` to disallow full screen display for the modal.
-   `true` to override `height` and `width`, and to display the modal in full screen.

Default: `false` |

`Promise`<`void`\>

___

▸ **closeModal**(): `Promise`<`void`\>

Closes an open modal on the current board.

The method doesn't take any arguments.

ℹ️ Note:

-   To close an open modal, `closeModal` must execute after [`openModal`](https://developers.miro.com/docs/ui_boardui#openmodal).
-   The app that closes the modal with `closeModal` must be the same that opened it with [`openModal`](https://developers.miro.com/docs/ui_boardui#openmodal).

Example:

```
// Open a modal
await miro.board.ui.openModal({
  url: 'show-this-page-in-the-modal.html',
  height: 400,
  width: 600,
  fullscreen: false,
});

// Close the open modal
await miro.board.ui.closeModal();
```

`Promise`<`void`\>

• **on**: (`event`: [`EventType`](https://developers.miro.com/docs/sdk_types_dist#eventtype), `handler`: (`event`: [`EventPayloads`](https://developers.miro.com/docs/sdk_types_dist#eventpayloads)) => `void`) => `void`(`event`: `"drop"`, `handler`: (`event`: [`DropEvent`](https://developers.miro.com/docs/sdk_types_dist#dropevent)) => `void`) => `void`(`event`: `"icon:click"`, `handler`: () => `void`) => `void`(`event`: `"app_card:open"`, `handler`: (`event`: [`AppCardOpenEvent`](https://developers.miro.com/docs/sdk_types_dist#appcardopenevent)) => `void`) => `void`(`event`: `"app_card:connect"`, `handler`: (`event`: [`AppCardConnectEvent`](https://developers.miro.com/docs/sdk_types_dist#appcardconnectevent)) => `void`) => `void`(`event`: `"selection:update"`, `handler`: (`event`: [`SelectionUpdateEvent`](https://developers.miro.com/docs/sdk_types_dist#selectionupdateevent)) => `void`) => `void`(`event`: `"online_users:update"`, `handler`: (`event`: [`OnlineUsersUpdateEvent`](https://developers.miro.com/docs/sdk_types_dist#onlineusersupdateevent)) => `void`) => `void`(`event`: `"items:create"`, `handler`: (`event`: [`ItemsCreateEvent`](https://developers.miro.com/docs/sdk_types_dist#itemscreateevent)) => `void`) => `void`(`event`: `"experimental:items:update"`, `handler`: (`event`: [`ItemsUpdateEvent`](https://developers.miro.com/docs/sdk_types_dist#itemsupdateevent)) => `void`) => `void`(`event`: `"items:delete"`, `handler`: (`event`: [`ItemsDeleteEvent`](https://developers.miro.com/docs/sdk_types_dist#itemsdeleteevent)) => `void`) => `void`

▸ (`event`, `handler`): `void`

```
miro.board.ui.on(event, handler);
```

If you want your app to react to an event by executing a function, you can use the `on` property to subscribe to events.  
The `on` property subscribes the app to listen to an event. When the event fires, the event handler executes a function to perform an action.

To subscribe to an event and its handler, pass to the `on` property:

-   The event that your app should listen to.
-   The event handler that the app needs to call when the event fires.

💡 To unsubscribe from an event and its handler, use the [`off`](https://developers.miro.com/docs/ui_boardui#off) property.

| Event type | Event handler | Event fires when... |
| --- | --- | --- |
| [`drop`](https://developers.miro.com/docs/ui_boardui#drop-event) | `(event: DropEvent)` | An HTML element is dropped from an open panel to the board. See also [Add drag and drop to your app](https://developers.miro.com/docs/add-drag-and-drop-to-your-app). |
| [`icon:click`](https://developers.miro.com/docs/ui_boardui#iconclick-event) | `()` | An app icon is clicked. See also [Add clicking on an icon to your app](https://developers.miro.com/docs/add-icon-click-to-your-app). |
| [`app_card:open`](https://developers.miro.com/docs/ui_boardui#app_cardopen-event) | `(event: AppCardOpenEvent)` | The app card status icon is [`connected`](https://developers.miro.com/docs/appcard_appcard-1#status), the icon is clicked, and it enables opening a modal with the app card detail view. |
| [`app_card:connect`](https://developers.miro.com/docs/ui_boardui#app_cardconnect-event) | `(event: AppCardConnectEvent)` | The app card status icon is [`disconnected`](https://developers.miro.com/docs/appcard_appcard-1#status), the icon is clicked, and it enables connecting an app card with a corresponding data source. |
| [`selection:update`](https://developers.miro.com/docs/ui_boardui#selectionupdate-event) | `(event: SelectionUpdateEvent)` | An area on the board is selected. |
| [`online_users:update`](https://developers.miro.com/docs/ui_boardui#online_usersupdate-event) | `(event: OnlineUsersUpdateEvent)` | The number of currently online board users changes. |
| [`items:create`](https://developers.miro.com/docs/ui_boardui#itemscreate-event) | `(event: ItemsCreateEvent)` | New items are created on the board. |
| [`experimental:items:update`](https://developers.miro.com/docs/ui_boardui#experimentalitemsupdate-event) | `(event: ItemsUpdateEvent)` | Existing items are updated on the board. |
| [`items:delete`](https://developers.miro.com/docs/ui_boardui#itemsdelete-event) | `(event: ItemsDeleteEvent)` | Existing items are deleted from the board. |

In general, when an app subscribes to an event, the event is dispatched to all [iframes](https://developers.miro.com/docs/app-panels-and-modals#iframes):

-   [Headless (main iframe)](https://developers.miro.com/docs/app-panels-and-modals#headless)
-   [Panel](https://developers.miro.com/docs/app-panels-and-modals#panel)
-   [Modal](https://developers.miro.com/docs/app-panels-and-modals#modal)

This behavior makes it easy to subscribe to an event from any of these iframes, without worrying about which iframe the event is dispatched to.

ℹ️ Note:

-   `icon:click` is dispatched only to the headless/main iframe.  
    Typically, [`icon:click`](https://developers.miro.com/docs/add-icon-click-to-your-app) is used to open a panel or a modal when a user clicks the [app icon](https://developers.miro.com/docs/add-a-logo-to-your-app) on the app toolbar or the app toolbar panel.

| Event type | Event dispatched to... |
| --- | --- |
| [`drop`](https://developers.miro.com/docs/ui_boardui#drop-event) | 
-   Headless (main iframe)
-   Panel
-   Modal

 |
| [`icon:click`](https://developers.miro.com/docs/ui_boardui#iconclick-event) | 

-   Headless (main iframe)

 |
| [`app_card:open`](https://developers.miro.com/docs/ui_boardui#app_cardopen-event) | 

-   Headless (main iframe)
-   Panel
-   Modal

 |
| [`app_card:connect`](https://developers.miro.com/docs/ui_boardui#app_cardconnect-event) | 

-   Headless (main iframe)
-   Panel
-   Modal

 |
| [`selection:update`](https://developers.miro.com/docs/ui_boardui#selectionupdate-event) | 

-   Headless (main iframe)
-   Panel
-   Modal

 |
| [`items:create`](https://developers.miro.com/docs/ui_boardui#itemscreate-event) | 

-   Headless (main iframe)
-   Panel
-   Modal

 |
| [`experimental:items:update`](https://developers.miro.com/docs/ui_boardui#experimentalitemsupdate-event) | 

-   Headless (main iframe)
-   Panel
-   Modal

 |
| [`items:delete`](https://developers.miro.com/docs/ui_boardui#itemsdelete-event) | 

-   Headless (main iframe)
-   Panel
-   Modal

 |

`void`

▸ (`event`, `handler`): `void`

```
on(event: 'drop', handler: (event: DropEvent) => void): void
```

When an app subscribes to this event, it's [dispatched to all iframes](https://developers.miro.com/docs/ui_boardui#dispatching-events).

`drop` registers the event associated with dropping one or more selected items on the board.

The event handler has the following properties:

| Property | Type | Description |
| --- | --- | --- |
| `x` | Number | Coordinate that defines the horizontal position of the HTML element to drop on the board. |
| `y` | Number | Coordinate that defines the vertical position of the HTML element to drop on the board. |
| `target` | `Element` interface | The HTML element that is the recipient of the `drop` event. |

Example:

```
/** When the selected HTML element is dropped on the board:
 *  1. A sticky note is created.
 *  2. The text of the HTML element is assigned to the
 *     'content' property of sticky note.
 */
miro.board.ui.on('drop', async ({x, y, target}) => {
  await miro.board.createStickyNote({
    x,
    y,
    content: target.innerText,
  });
});
```

| Name | Type |
| --- | --- |
| `event` | `"drop"` |
| `handler` | (`event`: [`DropEvent`](https://developers.miro.com/docs/sdk_types_dist#dropevent)) => `void` |

`void`

▸ (`event`, `handler`): `void`

```
on(event: 'icon:click', handler: () => void) => void
```

When an app subscribes to this event, it's [dispatched only to the headless/main iframe](https://developers.miro.com/docs/ui_boardui#dispatching-events).

`icon:click` registers click events on icons.  
When a user clicks an icon, the registered event handler for `icon:click` is called.  
Typically, `icon:click` is used to [open a panel](https://developers.miro.com/docs/ui_boardui#openpanel), or to [display a modal](https://developers.miro.com/docs/ui_boardui#openmodal).

Example:

```
/** When a user clicks the icon:
 *  1. The openPanel method is called
 *  2. The method opens the HTML page: `panel.html`
 */
miro.board.ui.on('icon:click', async () => {
  await miro.board.ui.openPanel({
    url: 'panel.html',
  });
});
```

| Name | Type |
| --- | --- |
| `event` | `"icon:click"` |
| `handler` | () => `void` |

`void`

▸ (`event`, `handler`): `void`

```
on(event: 'app_card:open', handler: (event: AppCardOpenEvent) => void): void;
```

When an app subscribes to this event, it's [dispatched to all iframes](https://developers.miro.com/docs/ui_boardui#dispatching-events).

`app_card:open` registers click events to open an app card from compact to detail view.

When a user clicks the icon to expand an app card to view it in detail, the registered event handler for `app_card:open` is called.  
Typically, `app_card:open` is used to [open a modal](https://developers.miro.com/docs/ui_boardui#openmodal) displaying the custom fields of the app card and their content.

Example:

```
/** When a user clicks the icon that expands an app card to view it in detail:
 *  1. The 'openModal' method is called
 *  2. The method opens a modal to display the specific app card content fetched from the URL
 */
// Listen to the 'app_card:open' event
miro.board.ui.on('app_card:open', (event) => {
  console.log('Subscribed to app card open event', event);
  const {appCard} = event;

  // Build a URL containing the app card ID.
  // You pass this URL with the 'openModal()' method below.
  // The code in the modal uses the 'appCardId' URL query parameter
  // to identify which app card was opened.
  const url = `https://my.app.example.com/modal.html?appCardId=${appCard.id}`;

  // Open the modal to display the content of the fetched app card
  miro.board.ui.openModal({
    url,
  });
});
```

`void`

▸ (`event`, `handler`): `void`

```
on(event: 'app_card:connect', handler: (event: AppCardConnectEvent) => void): void;
```

When an app subscribes to this event, it's [dispatched to all iframes](https://developers.miro.com/docs/ui_boardui#dispatching-events).

By default, newly created [app cards](https://developers.miro.com/docs/appcard_appcard-1) have a [`disconnected`](https://developers.miro.com/docs/appcard_appcard-1#status) status, unless the app card constructor sets a different value.  
To connect an app card to a corresponding data source in an external application, an app must listen to the `app_card:connect` event.

When an app listens to the `app_card:connect` event:

-   The `disconnected` icon ![App card status: disconnected](https://files.readme.io/0f2a313-appcard-status-disconnected.svg) is clickable.
-   On hovering over the icon, a tooltip is displayed to notify users that they can click the icon to connect the app card to a data source.
-   When users click the icon, the `app_card:connect` event fires.
-   The event handler needs to include at least the logic to:
    -   Retrieve the data source that the app card maps to.
    -   Sync data to populate the app card fields with any updated information.
    -   Update the app card status from `disconnected` to `connected`.
    -   [`sync()`](https://developers.miro.com/docs/board_board#sync) the app card to propagate the changes to the board.

If the app listens also to the [`app_card:open`](https://developers.miro.com/docs/ui_boardui#app_cardopen-event) event, it can react to it; typically, by [opening a modal](https://developers.miro.com/docs/ui_boardui#openmodal) to display the app card detail view.

Example:

```
/** In a typical flow:
 *  1. The app creates an app card.
 *     The 'disconnected' status icon is not clickable, yet.
 *  2. The app listens to the 'app_card:connect' event.
 *     The 'disconnected' status icon is clickable.
 *  3. The app listens to the 'app_card:open' event.
 *  4. When users click the 'disconnected' status icon, a tooltip prompts them to connect the app card to its data source.
 *  5. When the app card is connected, its status icon changes to 'connected'.
 *  6. Now the app can open a modal to display the app card detail view.
 */

// Create an app card
const appCard = await miro.board.createAppCard({
  title: 'This is the title of the app card',
  // Default status of new app cards
  status: 'disconnected',
});

// Listen to the 'app_card:connect' event
miro.board.ui.on('app_card:connect', (event) => {
  console.log('Connect the app card to its data source');
  const appcard = event.appCard;
  // Update the app card status to 'connected'
  appcard.status = 'connected';
  // Propagate the app card updates to the board
  appcard.sync();
});

// Listen to the 'app_card:open' event
miro.board.ui.on('app_card:open', (event) => {
  console.log('Subscribed to app card open event', event);
  // URL containing the app card ID.
  // The content is displayed inside the modal
  const url = `https://my.app.example.com/modal.html?appCardId=${appCard.id}`;
  // Open the modal to display the content of the fetched app card
  miro.board.ui.openModal({
    url,
  });
});
```

![](https://files.readme.io/cdf7416-appcard-disconnected.png)  
**Figure 1.** Newly created card, or duplicate app card through manual copying and pasting on the board UI. The app card status is disconnected. The tooltip notifies about the missing connection. The status icon isn't clickable.

![](https://files.readme.io/97c2707-appcard-connect.png)  
**Figure 2.** The app card status is `disconnected`. The tooltip prompts to connect it to its data source. The status icon is clickable.

![](https://files.readme.io/7e2966b-appcard-connected.png)  
**Figure 3.** The app card status is `connected`. The app listens to the [`app_card:open`](https://developers.miro.com/docs/ui_boardui#app_cardopen-event) event. When the icon is clicked, the event fires. Typically, it [opens a modal](https://developers.miro.com/docs/ui_boardui#openmodal) to display the app card detail view.

`void`

▸ (`event`, `handler`): `void`

```
on(event: 'selection:update', handler: (event: SelectionUpdateEvent) => void): void;
```

When an app subscribes to this event, it's [dispatched to all iframes](https://developers.miro.com/docs/ui_boardui#dispatching-events).

`selection:update` registers the event associated with updating the content of the current selection on the board.

When a user selects an area on the board, the registered event handler for `selection:update` is called.  
The event contains an array with the selected board items.  
If the selected area doesn't include any board items, the array is empty.

You can add logic to perform actions on the selection, such as filtering specific item types, and then modifying them.

Example:

```
/** When a user clicks and selects multiple board items on a board:
 *  1. The 'selection:update' method logs the selection to the developer console
 *  2. A filter identifies sticky note items in the selection
 *  3. The color of the sticky notes is changed to 'cyan'
 */

// Listen to the 'selection:update' event
miro.board.ui.on('selection:update', async (event) => {
  console.log('Subscribed to selection update event', event);
  console.log(event.items);
  const selectedItems = event.items;

  // Filter sticky notes from the selected items
  const stickyNotes = selectedItems.filter((item) => item.type === 'sticky_note');

  // Change the fill color of the sticky notes
  for (const stickyNote of stickyNotes) {
    stickyNote.style.fillColor = 'cyan';
    await stickyNote.sync();
  }
});
```

`void`

▸ (`event`, `handler`): `void`

```
on(event: 'online_users:update', handler: (event: OnlineUsersUpdateEvent) => void): void
```

When an app subscribes to this event, it's [dispatched to all iframes](https://developers.miro.com/docs/ui_boardui#dispatching-events).

`online_users:update` registers the event associated with a change in the number of users that are currently online on the board.

When a user joins or leaves the board, the registered event handler for `online_users:update` is called.  
The event contains an array with user IDs and names of the online users.

Your app can include logic to perform follow-up actions based on the change in online users.  
For example, it can greet a new user that just joined the board.

Example:

```
/** When the number of online users changes, identify the
 *  new online users and greet them with a notification.
 */

let currentOnlineUsers = [];
// Listen to the 'online_users:update' event.
await miro.board.ui.on('online_users:update', async (event) => {
  console.log('Subscribed to the update of online users');
  console.log('Online users: ', event.users);
  const onlineUsers = event.users;

  // Identify the new online users.
  const newUsers = onlineUsers.filter((user) => !currentOnlineUsers.find((u) => u.id === user.id));
  console.log('New users:', newUsers);

  // Greet the new online users.
  for (const newUser of newUsers) {
    await miro.board.notifications.showInfo(`Hello, ${newUser.name}!`);
  }

  currentOnlineUsers = onlineUsers;
});
```

`void`

▸ (`event`, `handler`): `void`

```
on(event: 'items:create', handler: (event: ItemsCreateEvent) => void): void;
```

When an app subscribes to this event, it's [dispatched to all iframes](https://developers.miro.com/docs/ui_boardui#dispatching-events).

`items:create` registers the event associated with the creation of a new item on the board.

When a user creates a new item on a board, the registered event handler for `items:create` is called.  
The event contains an array with the created board items.

Your app can include logic to perform follow-up actions on the created items. For example, it can filter specific item types, and then process them by fetching or setting their properties.

ℹ️ Note:

-   When creating a new item by copy-pasting or by duplicating an existing one, `items:create` isn't triggered.

Example:

```
/** When a user creates a new item on a board:
 *  1. 'items:create' logs the created items to the developer console.
 *  2. In the group of created items, a filter identifies sticky notes.
 *  3. The color of the sticky notes is set to 'cyan'.
 */

// Listen to the 'items:create' event.
miro.board.ui.on('items:create', async (event) => {
  console.log('Subscribed to the creation of new board items', event);
  console.log(event.items);
  const createdItems = event.items;

  // Filter sticky notes from the created items.
  const stickyNotes = createdItems.filter((item) => item.type === 'sticky_note');

  // Change the fill color of the sticky notes.
  for (const stickyNote of stickyNotes) {
    stickyNote.style.fillColor = 'cyan';
    await stickyNote.sync();
  }
});
```

`void`

▸ (`event`, `handler`): `void`

 **![Experimental feature](https://files.readme.io/4598fa6-experimental-flask-color.svg) Experimental**

```
on(event: 'experimental:items:update', handler: (event: ItemsUpdateEvent) => void): void;
```

When an app subscribes to this event, it's [dispatched to all iframes](https://developers.miro.com/docs/ui_boardui#dispatching-events).

`experimental:items:update` registers the event associated with updating an item on the board.

When a user updates one or more items on the board, the registered event handler for `experimental:items:update` is called.  
The event contains an array with the updated board items.

Your app can include logic to perform follow-up actions on the updated items. For example, it can log a list of the updated board items.

ℹ️ Note:

-   Currently, `experimental:items:update` fires only when items are _moved_ on the board, when an item dimensions are _resized_, when an item is _rotated_ on the board, and when updating the _scale_ of an item.  
    In the future, the event will also fire when updating other item data.

Example:

```
/**
 * When a user updates one or more items on the board:
 * 'experimental:items:update' logs all the updated board items to the developer console.
 */

// Listen to the 'experimental:items:update' event.
miro.board.ui.on('experimental:items:update', async (event) => {
  console.log('Subscribed to updates of board items', event);
  console.log('Updated items: ', event.items);
});
```

| Name | Type |
| --- | --- |
| `event` | `"experimental:items:update"` |
| `handler` | (`event`: [`ItemsUpdateEvent`](https://developers.miro.com/docs/sdk_types_dist#itemsupdateevent)) => `void` |

`void`

▸ (`event`, `handler`): `void`

```
on(event: 'items:delete', handler: (event: ItemsDeleteEvent) => void): void;
```

When an app subscribes to this event, it's [dispatched to all iframes](https://developers.miro.com/docs/ui_boardui#dispatching-events).

`items:delete` registers the event associated with deleting an item from the board.

When a user deletes one or more items from the board, the registered event handler for `items:delete` is called.  
The event contains an array with the deleted board items.

Your app can include logic to perform follow-up actions on the deleted items. For example, it can log a list of the deleted board items.

Example:

```
/**
 * When a user deletes one or more items from the board:
 * 'items:delete' logs all deleted board items to the developer console.
 */

// Listen to the 'items:delete' event.
miro.board.ui.on('items:delete', async (event) => {
  console.log('Subscribed the deletion of board items', event);
  console.log('Deleted items: ', event.items);
});
```

`void`

___

• **off**: (`event`: [`EventType`](https://developers.miro.com/docs/sdk_types_dist#eventtype), `handler`: (`event`: [`EventPayloads`](https://developers.miro.com/docs/sdk_types_dist#eventpayloads)) => `void`) => `void`(`event`: `"drop"`, `handler`: (`event`: [`DropEvent`](https://developers.miro.com/docs/sdk_types_dist#dropevent)) => `void`) => `void`(`event`: `"icon:click"`, `handler`: () => `void`) => `void`(`event`: `"app_card:open"`, `handler`: (`event`: [`AppCardOpenEvent`](https://developers.miro.com/docs/sdk_types_dist#appcardopenevent)) => `void`) => `void`(`event`: `"app_card:connect"`, `handler`: (`event`: [`AppCardConnectEvent`](https://developers.miro.com/docs/sdk_types_dist#appcardconnectevent)) => `void`) => `void`(`event`: `"selection:update"`, `handler`: (`event`: [`SelectionUpdateEvent`](https://developers.miro.com/docs/sdk_types_dist#selectionupdateevent)) => `void`) => `void`(`event`: `"online_users:update"`, `handler`: (`event`: [`OnlineUsersUpdateEvent`](https://developers.miro.com/docs/sdk_types_dist#onlineusersupdateevent)) => `void`) => `void`(`event`: `"items:create"`, `handler`: (`event`: [`ItemsCreateEvent`](https://developers.miro.com/docs/sdk_types_dist#itemscreateevent)) => `void`) => `void`(`event`: `"experimental:items:update"`, `handler`: (`event`: [`ItemsUpdateEvent`](https://developers.miro.com/docs/sdk_types_dist#itemsupdateevent)) => `void`) => `void`(`event`: `"items:delete"`, `handler`: (`event`: [`ItemsDeleteEvent`](https://developers.miro.com/docs/sdk_types_dist#itemsdeleteevent)) => `void`) => `void`

▸ (`event`, `handler`): `void`

```
miro.board.ui.off(event, handler);
```

When an app no longer needs to listen to an event to trigger an event handler, it can use the `off` property to unsubscribe from it.  
To unsubscribe from an event and its handler, pass to the `off` property:

-   The event whose handler you want your app to unsubscribe from.
-   The event handler that you previously registered with the `on` property, and that your app no longer needs to listen to.

💡 To subscribe to an event and its handler, use the [`on`](https://developers.miro.com/docs/ui_boardui#on) property.

See the [supported events for the `on` property](https://developers.miro.com/docs/ui_boardui#supported-events).

`void`

▸ (`event`, `handler`): `void`

```
off(event: 'drop', handler: (event: DropEvent) => void): void
```

Enables unsubscribing from a [`drop`](https://developers.miro.com/docs/ui_boardui#drop-event) event handler.  
To unsubscribe, pass to the `off` property:

-   The `drop` event.
-   The event handler that you previously registered with the `on` property, and that your app no longer needs to listen to.

Example:

```
// Add a 'drop' event handler to drag and drop images.
const drop = async (event: DropEvent) => {
  const {x, y, target} = event;
  if (target instanceof HTMLImageElement) {
    const image = await miro.board.createImage({
      x,
      y,
      url: target.src,
    });
  }
};

// Register the 'drop' event so that the app listens to it.
miro.board.ui.on('drop', drop);

// Unsubscribe from the 'drop' handler.
// The app no longer creates image items on drag and drop.
miro.board.ui.off('drop', drop);
```

| Name | Type |
| --- | --- |
| `event` | `"drop"` |
| `handler` | (`event`: [`DropEvent`](https://developers.miro.com/docs/sdk_types_dist#dropevent)) => `void` |

`void`

▸ (`event`, `handler`): `void`

```
off(event: 'icon:click', handler: () => void) => void
```

Enables unsubscribing from an [`icon:click`](https://developers.miro.com/docs/ui_boardui#iconclick-event) event handler.  
To unsubscribe, pass to the `off` property:

-   The `icon:click` event.
-   The event handler that you previously registered with the `on` property, and that your app no longer needs to listen to.

Example:

```
// Add an 'iconClick' event handler to open a panel upon clicking an icon.
const iconClick = async () => {
  await miro.board.ui.openPanel({
    url: 'panel.html',
  });
};

// Register the 'icon:click' event so that the app listens to it.
miro.board.ui.on('icon:click', iconClick);

// Unsubscribe from the 'icon:click' event handler.
// The app no longer enables opening a panel when clicking an icon.
miro.board.ui.off('icon:click', iconClick);
```

| Name | Type |
| --- | --- |
| `event` | `"icon:click"` |
| `handler` | () => `void` |

`void`

▸ (`event`, `handler`): `void`

```
off(event: 'app_card:open', handler: (event: AppCardOpenEvent) => void): void;
```

Enables unsubscribing from an [`app_card:open`](https://developers.miro.com/docs/ui_boardui#app_cardopen-event) event handler.  
To unsubscribe, pass to the `off` property:

-   The `app_card:open` event.
-   The event handler that you previously registered with the `on` property, and that your app no longer needs to listen to.

Example:

```
// Create an app card.
const appCard = await miro.board.createAppCard({
  title: 'This is the title of the app card',
  status: 'disconnected',
});

// Add an 'appCardOpen' event handler for the 'app_card:open' event.
const appCardOpen = async (event: AppCardOpenEvent) => {
  const appcard = event.appCard;
  const url = `https://my.app.example.com/modal.html?appCardId=${appCard.id}`;
  miro.board.ui.openModal({
    url,
  });
};

// Register the 'app_card:open' event so that the app listens to it.
miro.board.ui.on('app_card:open', appCardOpen);

// Unsubscribe from the 'app_card:open' event handler.
// The app no longer enables opening a modal with the detail view of an app card.
miro.board.ui.off('app_card:open', appCardOpen);
```

`void`

▸ (`event`, `handler`): `void`

```
off(event: 'app_card:connect', handler: (event: AppCardConnectEvent) => void): void;
```

Enables unsubscribing from the [`app_card:connect`](https://developers.miro.com/docs/ui_boardui#app_cardconnect-event) event and its handler.  
To unsubscribe, pass to the `off` property:

-   The `app_card:connect` event.
-   The event handler that you previously registered with the `on` property, and that your app no longer needs to listen to.

Example:

```
// Create an app card.
const appCard = await miro.board.createAppCard({
  title: 'This is the title of the app card',
  status: 'disconnected',
});

// Add an 'appCardConnect' event handler for the 'app_card:connect' event.
const appCardConnect = async (event: AppCardConnectEvent) => {
  const appcard = event.appCard;
  appcard.status = 'connected';
  appcard.sync();
};

// Register the 'app_card:connect' event so that the app listens to it.
miro.board.ui.on('app_card:connect', appCardConnect);

// Unsubscribe from the 'app_card:connect' event handler.
// The app no longer enables connecting an app card to a data source.
miro.board.ui.off('app_card:connect', appCardConnect);
```

`void`

▸ (`event`, `handler`): `void`

```
off(event: 'selection:update', handler: (event: SelectionUpdateEvent) => void): void;
```

Enables unsubscribing from a [`selection:update`](https://developers.miro.com/docs/ui_boardui#selectionupdate-event) event handler.  
To unsubscribe, pass to the `off` property:

-   The `selection:update` event.
-   The event handler that you previously registered with the `on` property, and that your app no longer needs to listen to.

Example:

```
// Add a 'selectionUpdate' event handler to update the color of sticky notes included in a selection.
const selectionUpdate = async (event: SelectionUpdateEvent) => {
  const selectedItems = event.items;
  const stickyNotes = selectedItems.filter((item) => item.type === 'sticky_note');
  for (const stickyNote of stickyNotes) {
    stickyNote.style.fillColor = 'cyan';
    await stickyNote.sync();
  }
};

// Register the 'selection:update' event so that the app listens to it.
miro.board.ui.on('selection:update', selectionUpdate);

// Unsubscribe from the 'selection:update' event handler.
// The app no longer enables updating the color of the sticky notes included in a selection to cyan.
miro.board.ui.off('selection:update', selectionUpdate);
```

`void`

▸ (`event`, `handler`): `void`

```
off(event: 'online_users:update', handler: (event: OnlineUsersUpdateEvent) => void): void
```

Enables unsubscribing from an [`online_users:update`](https://developers.miro.com/docs/ui_boardui#online_usersupdate-event) event handler.  
To unsubscribe, pass to the `off` property:

-   The `online_users:update` event.
-   The event handler that you previously registered with the `on` property, and that your app no longer needs to listen to.

Example:

```
// Add an 'online_users:update' event handler to greet new online users joining the board.
let currentOnlineUsers = [];
const onlineUsersUpdate = async (event) => {
  const onlineUsers = event.users;
  const newUsers = onlineUsers.filter((user) => !currentOnlineUsers.find((u) => u.id === user.id));

  for (const newUser of newUsers) {
    await miro.board.notifications.showInfo(`Hello, ${newUser.name}!`);
  }

  currentOnlineUsers = onlineUsers;
};

// Register the 'online_users:update' event so that the app listens to it.
await miro.board.ui.on('online_users:update', onlineUsersUpdate);

// Unsubscribe from the 'online_users:update' event handler.
// The app no longer greets new online users.
await miro.board.ui.off('online_users:update', onlineUsersUpdate);
```

`void`

▸ (`event`, `handler`): `void`

```
off(event: 'items:create', handler: (event: ItemsCreateEvent) => void): void;
```

Enables unsubscribing from an [`items:create`](https://developers.miro.com/docs/ui_boardui#itemscreate-event) event handler.  
To unsubscribe, pass to the `off` property:

-   The `items:create` event.
-   The event handler that you previously registered with the `on` property, and that your app no longer needs to listen to.

Example:

```
// Add an 'itemsCreate' event handler to update the color of user-created sticky notes.
const itemsCreate = async (event: ItemsCreateEvent) => {
  const createdItems = event.items;
  const stickyNotes = createdItems.filter((item) => item.type === 'sticky_note');
  for (const stickyNote of stickyNotes) {
    stickyNote.style.fillColor = 'cyan';
    await stickyNote.sync();
  }
};

// Register the 'items:create' event so that the app listens to it.
miro.board.ui.on('items:create', itemsCreate);

// Unsubscribe from the 'items:create' event handler.
// The app no longer updates the color of user-created sticky notes.
miro.board.ui.off('items:create', itemsCreate);
```

`void`

▸ (`event`, `handler`): `void`

 **![Experimental feature](https://files.readme.io/4598fa6-experimental-flask-color.svg) Experimental**

```
off(event: 'experimental:items:update', handler: (event: ItemsUpdateEvent) => void): void;
```

Enables unsubscribing from an [`experimental:items:update`](https://developers.miro.com/docs/ui_boardui#itemsupdate-event) event handler.  
To unsubscribe, pass to the `off` property:

-   The `experimental:items:update` event.
-   The event handler that you previously registered with the `on` property, and that your app no longer needs to listen to.

Example:

```
// Add an 'itemsUpdate' event handler to log user-updated board items to the developer console.
const itemsUpdate = async (event: ItemsUpdateEvent) => {
  console.log(event.items);
};

// Register the 'experimental:items:update' event so that the app listens to it.
miro.board.ui.on('experimental:items:update', itemsUpdate);

// Unsubscribe from the 'experimental:items:update' event handler.
// The app no longer enables logging user-updated board items.
miro.board.ui.off('experimental:items:update', itemsUpdate);
```

| Name | Type |
| --- | --- |
| `event` | `"experimental:items:update"` |
| `handler` | (`event`: [`ItemsUpdateEvent`](https://developers.miro.com/docs/sdk_types_dist#itemsupdateevent)) => `void` |

`void`

▸ (`event`, `handler`): `void`

```
off(event: 'items:delete', handler: (event: ItemsDeleteEvent) => void): void;
```

Enables unsubscribing from an [`items:delete`](https://developers.miro.com/docs/ui_boardui#itemscreate-event) event handler.  
To unsubscribe, pass to the `off` property:

-   The `items:delete` event.
-   The event handler that you previously registered with the `on` property, and that your app no longer needs to listen to.

Example:

```
// Add an 'itemsDelete' event handler to log user-deleted board items to the developer console.
const itemsDelete = async (event: ItemsDeleteEvent) => {
  console.log(event.items);
};

// Register the 'items:delete' event so that the app listens to it.
miro.board.ui.on('items:delete', itemsDelete);

// Unsubscribe from the 'items:delete' event handler.
// The app no longer enables logging user-deleted board items.
miro.board.ui.off('items:delete', itemsDelete);
```

`void`

Updated 6 days ago

___

Go to `board.viewport` to learn more about the methods to set, get, and zoom to the viewport.

-   [Viewport](https://developers.miro.com/docs/viewport_viewport)

-   [Table of Contents](https://developers.miro.com/docs/ui_boardui#)
-   -   [Table of contents](https://developers.miro.com/docs/ui_boardui#table-of-contents)
        -   [Methods](https://developers.miro.com/docs/ui_boardui#methods)
        -   [Properties](https://developers.miro.com/docs/ui_boardui#properties)
    -   [Methods](https://developers.miro.com/docs/ui_boardui#methods-1)
        -   [openPanel](https://developers.miro.com/docs/ui_boardui#openpanel)
        -   [closePanel](https://developers.miro.com/docs/ui_boardui#closepanel)
        -   [openModal](https://developers.miro.com/docs/ui_boardui#openmodal)
        -   [closeModal](https://developers.miro.com/docs/ui_boardui#closemodal)
    -   [Properties](https://developers.miro.com/docs/ui_boardui#properties-1)
        -   [on](https://developers.miro.com/docs/ui_boardui#on)
        -   [off](https://developers.miro.com/docs/ui_boardui#off)