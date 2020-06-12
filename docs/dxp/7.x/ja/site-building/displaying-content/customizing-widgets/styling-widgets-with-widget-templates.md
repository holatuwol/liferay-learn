# Styling Widgets with Widget Templates

Widget templates are used to customize the appearance and functionality of various existing widgets. Creating a widget template allows you to use a script to adapt the widget appearance and functionality. You can create a widget template for many widgets available out-of-the-box with Liferay, such as the *Asset Publisher* and *Media Gallery* widgets.

## Creating a Widget Template

Follow these steps to create a widget template

1.  From the Product Menu, click *Site Builder* → *Widget Templates.*

    ![The Widget Template page.](./styling-widgets-with-widget-templates/images/01.png)

2.  Click the *Add* (![Add icon](../../../images/icon-add.png)) button, then select which widget to create a widget template for (e.g., *Asset Publisher*).

    ![The Widget Template creation page.](./styling-widgets-with-widget-templates/images/02.png)

3.  Add a name for your widget template.

4.  Under the *Script* section, click into the body of the template editor.

5.  Click one of the fields in the *Fields* menu to insert that field into the editor. The value of the field will be displayed when your widget template is used.

    ``` note::
       Clicking any of the fields in the menu will insert the field name wherever the cursor is. You can also begin typing a field name yourself (starting with "${") to show suggestions for auto-completion.
    ```

6.  Click *Save* to complete the template.

## Applying a Widget Template

Once you have a widget template ready for a widget on one of your pages, follow these steps:

1.  Click the Actions (![Actions icon](../../../images/icon-actions.png)) icon on the appropriate widget, then click *Configuration.*

2.  Find the Display Template drop-down menu, and select the name of the desired template.

    ![The Display Template drop-down box in the Asset Publisher configuration.](./styling-widgets-with-widget-templates/images/03.png)

    ``` note::
       This drop-down menu is on the default tab, Setup, for most widgets. For the Asset Publisher widget, this setting is found under the Display Settings tab.
    ```

3.  Click *Save.*
    
     <!-- screenshot -->

The widget has been modified to use a custom appearance. By using more advanced widget template scripts, you can further enhance the capabilities of your widgets.
