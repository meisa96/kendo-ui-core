---
title: Common Issues
page_title: Common Issues | Kendo UI Troubleshooting
description: "Learn about the solutions of issues that may occur while working with Kendo UI controls."
previous_url: /troubleshooting
slug: troubleshooting_common_issues_kendoui
---

# Common Issues

This page provides solutions for common problems you may encounter while working with Kendo UI widgets.

## Offline Examples

### Examples Fail to Load Sample Data

The most common cause for this error is loading the demos from the file system. Google Chrome, for example, will disallow accessing the JSON files needed for the demos to run.

**Solution**

Host the offline demos on a web server and load them from there.

## jQuery

### jQuery Is Unavailable or Undefined

If jQuery is not included, or is included after the Kendo UI JavaScript files, or is included after Kendo UI widget initialization statements, the Kendo UI widgets will not function as expected.

The following JavaScript errors will be thrown (depending on the browser):

* `ReferenceError: jQuery is not defined` (in Google Chrome and Firefox)
* `jQuery is undefined` (in Internet Explorer)

**Solution**

Make sure that jQuery is included only before the Kendo UI JavaScript files and before any JavaScript statements that depend on it.

## Widgets

### Widgets Are Unavailable or Undefined

If jQuery is included more than once in the page all existing jQuery plugins (including Kendo UI) will be wiped out. Will also occur if the [required jQuery file]({% slug jquerysupport_kendoui %}) is not included.

Depending on the browser, the following JavaScript errors will be thrown:

* TypeError: Object `#<Object>` has no method `kendoGrid` (in Google Chrome)
* TypeError: `$("#Grid").kendoGrid` is not a function (in Firefox)
* Object does not support property or method `kendoGrid` (in Internet Explorer 9 and later)
* Object does not support this property or method (in older versions of Internet Explorer)

> **Important**
>
> Not just the Kendo UI Grid, but all Kendo UI widgets are affected by this issue with only the error message being different. For example, `kendoChart is not a function` or `Object has no method kendoEditor`.

**Solution**

Make sure jQuery is not included more than once in your page. Remove any duplicate `script` references to jQuery. Include all [required Kendo JavaScript files]({% slug jquerysupport_kendoui %}).

### Widgets Cannot Be Initialized in Internet Explorer Compatibility Mode

Kendo widgets provide a WAI-ARIA support, which means that some ARIA-specific attributes are added to the HTML element. When a widget tries to add an ARIA attribute using [jQuery's `attr` method](http://api.jquery.com/attr/), which in turn calls the [`Element.setAttribute` method](https://developer.mozilla.org/en-US/docs/Web/API/Element/setAttribute), the Internet Explorer in Compatibility mode will raise a JavaScript error with the following message:

* SCRIPT3: Member not found (in Internet Explorer 10+ in Compatibility Mode)

The problem is reported to Microsoft on `https://connect.microsoft.com/IE/feedback/details/774078`. Also there is a [jQuery bug report](http://bugs.jquery.com/ticket/12577)
where more information can be found.

> **Important**
>
> All Kendo widgets, which add ARIA attributes to HTML elements, will be affected.

**Solution**

**Option 1** Force the Internet Explorer to use the Edge mode:

 ```
 <meta http-equiv="X-UA-Compatible" content="IE=edge,chrome=1" />
 ```

**Option 2** Path jQuery. You can find more information about the possible path in the aforementioned jQuery bug report.

### Unknown DataSource transport type json Warning Is Displayed

The `Unknown DataSource transport type json` warning might be caused by either of the following reasons:

* Invalid Kendo UI DataSource [`type`](/api/javascript/data/datasource/configuration/type) configuration is set. For example:

    ```
    dataSource: {
        type: "json"
    }
    ```

* A JavaScript file is missing when using a DataSource type that is not included in `kendo.all.min.js`. For example, `aspnetmvc-ajax`, `jsdo`, and others. Specifically, `json` is not a valid DataSource `type` and it does not require a separate JavaScript file.

**Solution**

Use a valid `type` value, or remove the `type` property, or add the corresponding missing file&mdash;for example, `kendo.aspnetmvc.min.js` when using the Kendo UI MVC wrappers.

Note that the [dataSource `type`](/api/javascript/data/datasource/configuration/type) differs from the [`type` of the transport actions](/api/javascript/data/datasource/configuration/transport.read.type).

### Widget Throws the e.slice is not a function Error

The `Uncaught TypeError: e.slice is not a function` error indicates that the response which is received from the remote data source is not an array while the widget expects a simple array for its data source.

Widgets like the TreeView or the MultiSelect need only a simple array while the Grid needs an envelope with additional information such as total, errors, and aggregates. For more information on what information each widget expects, review the demo of the respective control.

The possible causes for the `e.slice is not a function` error can be any or a combination of the following:

**Cause 1**

The server does not return an actual list of objects but empty data, an error response, or a single item. In such cases, you get a single object or HTML instead of a serialized array.

**Solution 1**

Step through the server method that returns data and monitor the response in the browser dev toolbar to see what you get and ensure it is something like `[{"fieldName": 123, "otherField": "someValue"}, {"fieldName": 234, "otherField": "otherValue"}]`.

**Cause 2**

Under MVC, the server method (action) often uses `myData.ToDataSourceResult(request)` unnecessarily and wraps the data in an envelope similar to `{data: [the array from your data goes here], total: <the count goes here> }`. In such cases, the widget is not able to get the array at the root level as it expects.

**Solution 2**

Use either of the following suggestions:

* Apply the same approach as the Kendo UI demos demonstrate&mdash;return the data array only. For example, in an MVC action, use the `return Json(myData, JsonRequestBehavior.AllowGet);` configuration without`.ToDataSourceResul()` as demonstrated in the [MVC TreeView Remote Binding](https://demos.telerik.com/aspnet-mvc/treeview/remote-data-binding) demo.
* By configuring its data source `Schema`, indicate to the widget which fields you expect it look for.

        // C#
        .Schema(schema =>
          {
              schema.Data("Data")
                 .Total("Total");
          }


### Input Widgets Do Not Raise Change Event When API Is Used

The change event of an input widget is triggered only by user action. DOM elements work in the same way. If you need to trigger an event manually use the [trigger method](/api/javascript/ui/widget/methods/trigger).

### Creating Multiple Widgets Throws JavaScript Errors

This will happen if two or more widgets are initialized from elements that have same IDs. jQuery will find only the first one every time it searches for it and thus try to initialize the first element in the DOM multiple times.

**Solution**

The ID for each element on the page should be unique.

	<textarea id="editor"></textarea>
	<textarea id="editor"></textarea>
	<script>
		$('#editor').kendoEditor();
		$('#editor').kendoEditor(); // problem
	</script>

### When Creating Multiple Widgets Only One of Them Works

For a solution, see the [Creating Multiple Widgets Throws JavaScript Errors](#creating-multiple-widgets-throws-javascript-errors) section above.

### Third-Party Modal Popups Prevent Access to Kendo UI Widgets

Some third-party modal popups prevent access to focusable elements, which are rendered outside the modal popup. Such widgets include the Bootstrap modal dialog, the jQuery UI modal dialog, and possibly others. The Kendo UI widgets, which are affected by this behavior are all widgets, which render their own detached popups, e.g. AutoComplete, ColorPicker, ComboBox, DropDownList, DateTimePicker, Editor, Grid, and MultiSelect. The popups of these Kendo UI widgets are rendered as children of the `<body>` and as a result, the third-party modal popup will prevent access to them.

**Solution**

There are two ways to avoid this problem:

* Disable the modal popup's modality, so that elements outside it can be focused.
* Use a [modal](/api/javascript/ui/window/configuration/modal) [Kendo UI Window]({% slug overview_kendoui_window_widget %}) instead of a third-party popup.

### Widgets Do Not Work Correctly on Touch Devices

Client libraries that interfere with touch events, such as FastClick, are not compatible with Kendo UI and may break the behavior of the widgets. For example, they may cause a drop-down list to close immediately after opening.

**Solution**

For more information on this issue, refer to [What Exactly Is... The 300ms Click Delay](http://www.telerik.com/blogs/what-exactly-is.....-the-300ms-click-delay).

### Widget Popup Is Offset Incorrectly in Internet Explorer

When Kendo UI is used with jQuery `1.12.0` or `2.2.0`, some issues with the popup positioning might occur. The popup is offset incorrectly when the page is scrolled.

**Solution**

The reason for this issue is a bug in the aforementioned jQuery version. According to jQuery's bug tracker, it will be addressed in the next patch release of jQuery. For more details, refer to [https://github.com/telerik/kendo-ui-core/issues/1375](https://github.com/telerik/kendo-ui-core/issues/1375).

### Incorrect Appearance or Errors Occur in Hidden Widgets

If you display widgets that have been in an initially hidden container, call their [`resize()`](/api/javascript/ui/widget/methods/resize) method after you show them. Initializing widgets on elements with the `style="display: none;"` configuration might cause errors, such as inability to calculate dimensions and positions or even throw errors. The reason for this behavior is that such calculations are not available for elements that are not rendered by the browser.

**Solution**

Usually, delaying the widget initialization until after it is displayed resolves the issue and improves the page performance.

## CDN

### Scripts or Stylesheets Do Not Load from CDN

For a solution, refer to [Kendo UI CDN Fallback and Troubleshooting]({% slug kendoui_cdn_services_installation %}#troubleshooting).

## Status Codes

### 404 Status Code Is Thrown for WOFF or TTF Font Files

For a solution, refer to [Serving Font Files]({% slug hybridiconfonts_hybridkendoui %}#serving-font-files).

### 404 Status Code Is Thrown for JSON Files on IIS

By default, IIS does not serve files with unknown extensions. The mime types can be specified either through the IIS management console (inetmgr) or in the site `Web.config`.

**Solution**

The example below demonstrates how to configure the IIS `Web.config`. Note the mime is removed first to avoid clashes if it is already defined.



```xml
    <?xml version="1.0"?>
    <configuration>
        ...
        <system.webServer>
            ...
            <staticContent>
                <remove fileExtension=".json" />
                <mimeMap fileExtension=".json" mimeType="application/json" />
            </staticContent>
        </system.webServer>
    </configuration>
```

## Ajax

### Widget Object Is Undefined after Loading a Page through AJAX

This issue is usually caused when the page loaded via AJAX contains a script reference to jQuery. When jQuery is re-initialized, all jQuery-based data attributes are cleared, including the data(`kendoWidget`) attribute that holds the Kendo UI widget object.

**Solution**

Load a partial HTML fragment that does not contain any unneeded jQuery references, or use an `iframe` to load the complete page.

The example below demonstrates a test issue.



    $("#dialog").kendoWinodow({
        // loads complete page
        content: "/foo"
    });

The example below demonstrates the solution of the test issue above.



    $("#dialog").kendoWindow({
        // load complete page...
        content: "/foo",
        // ... and show it in an iframe
        iframe: true
    });

    // or

    $("#dialog").kendoWinodow({
         // load partial page, without jQuery reference
        content: "/foo"
    });

## Styles and Themes

### Icons Disappear after Upgrade

As of the R1 2017 release, Kendo UI widgets utilize font-icons instead of sprites. This approach outdates some CSS, which are now removed from the built-in styles.

**Solution**

To provide for the backward compatibility of any custom icons you add to your project, include the following styles in your application:

```css
/* Provides backward compatibility support for custom sprites */
.k-sprite {
    display: inline-block;
    width: 16px;
    height: 16px;
    overflow: hidden;
    background-repeat: no-repeat;
    font-size: 0;
    line-height: 0;
    text-align: center;
    -ms-high-contrast-adjust: none;
}

/* Removes sprite styles from font icons */
.k-sprite.k-icon {
    font-size: 16px;
    line-height: 1;
}
```

## See Also

* [JavaScript Errors]({% slug troubleshooting_javascript_errors_kendoui %})
* [Performance Issues]({% slug troubleshooting_system_memory_symptoms_kendoui %})
* [Content Security Policy]({% slug troubleshooting_content_security_policy_kendoui %})
* [Common Issues in Kendo UI Excel Export]({% slug troubleshooting_excel_export_kendoui %})
* [Common Issues in Kendo UI Charts]({% slug troubleshooting_chart_widget %})
* [Performance Issues in Kendo UI Widgets for Data Visualization]({% slug tipsandtricks_kendouistyling %})
* [Common Issues in Kendo UI ComboBox]({% slug troubleshooting_common_issues_combobox_kendoui %})
* [Common Issues in Kendo UI Diagram]({% slug troubleshooting_diagram_widget %})
* [Common Issues in Kendo UI DropDownList]({% slug troubleshooting_common_issues_dropdownlist_kendoui %})
* [Common Issues in Kendo UI Editor]({% slug troubleshooting_editor_widget %})
* [Common Issues in Kendo UI MultiSelect]({% slug troubleshooting_common_issues_multiselect_kendoui %})
* [Common Issues in Kendo UI Scheduler]({% slug troubleshooting_scheduler_widget %})
* [Common Issues in Kendo UI Upload]({% slug troubleshooting_upload_widget %})
* [Common Issues in Telerik UI for ASP.NET MVC](http://docs.telerik.com/aspnet-mvc/troubleshoot/troubleshooting)
* [Validation Issues in Telerik UI for ASP.NET MVC](http://docs.telerik.com/aspnet-mvc/troubleshoot/troubleshooting-validation)
* [Scaffolding Issues in Telerik UI for ASP.NET MVC](http://docs.telerik.com/aspnet-mvc/troubleshoot/troubleshooting-scaffolding)
* [Common Issues in the Grid ASP.NET MVC HtmlHelper Extension](http://docs.telerik.com/aspnet-mvc/helpers/grid/troubleshoot/troubleshooting)
* [Excel Export with the Grid ASP.NET MVC HtmlHelper Extension](http://docs.telerik.com/aspnet-mvc/helpers/grid/troubleshoot/excel-export-issues)
* [Common Issues in the Spreadsheet ASP.NET MVC HtmlHelper Extension](http://docs.telerik.com/aspnet-mvc/helpers/spreadsheet/troubleshoot/troubleshooting)
* [Common Issues in the Upload ASP.NET MVC HtmlHelper Extension](http://docs.telerik.com/aspnet-mvc/helpers/upload/troubleshoot/troubleshooting)
