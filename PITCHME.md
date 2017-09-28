
# Layouts in Magento 2

---

## XML

*Magento layouts are built with XML*

- XML stands for eXtensible Markup Language.
- XML was designed to store and transport data.
- XML was designed to be both human- and machine-readable.

+++

### XML Example

```xml
<?xml version="1.0" encoding="UTF-8"?>
<note>
  <to>Tove</to>
  <from>Jani</from>
  <heading>Reminder</heading>
  <body>Don't forget me this weekend!</body>
</note>
```
@[1](Declare the file type)
@[2-7](Include the data)

[More examples](https://www.w3schools.com/xml/xml_examples.asp)

---

## Two layers of layouts

- **Page layout file** = a wireframe of the page
- **Page configuration file** = the detailed structure

---

### 1: Page layout files

- May be found in both modules and themes
- Most just use Magento's layouts: `2columns-left.xml`, `2columns-right.xml`, `1column.xml`, etc

Found here:
`<module_dir>/view/frontend/page_layout` *or*
`<theme_dir>/<Namespace>_<Module>/page_layout`

+++

![Page layout](http://devdocs.magento.com/common/images/layouts_containers_defn.jpg)

+++

All new page layouts must be declared!

`<module_dir>/view/frontend/layouts.xml`
`<theme_dir>/<Namespace>_<Module>/layouts.xml`

+++

### Structure and layout instructions allowed

 - `<container>`
 - `<referenceContainer>`
 - `<move>`
 - `<update>`

**...only!**

+++

#### Example, from M2's `2columns-left.xml`

```xml
<layout xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="urn:magento:framework:View/Layout/etc/page_layout.xsd">
    <update handle="1column"/>
    <referenceContainer name="columns">
        <container name="div.sidebar.main" htmlTag="div" htmlClass="sidebar sidebar-main" after="main">
            <container name="sidebar.main" as="sidebar_main" label="Sidebar Main"/>
        </container>
        <container name="div.sidebar.additional" htmlTag="div" htmlClass="sidebar sidebar-additional" after="div.sidebar.main">
            <container name="sidebar.additional" as="sidebar_additional" label="Sidebar Additional"/>
        </container>
    </referenceContainer>
</layout>
```

@[1,11]
@[2]
@[3-10]

---

### 2: Page Configuration files

- Found in both modules and themes.
- Adds content to the page layout. 
- Also contains page meta-information, and contents of the `<head>` section.

+++

![Page configuration blocks](http://devdocs.magento.com/common/images/layouts_block_defn.jpg)

+++

### Structure and layout instructions allowed

[There's a lot.](http://devdocs.magento.com/guides/v2.1/frontend-dev-guide/layouts/layout-types.html#Page-configuration-structure-and-allowed-layout-instructions)

+++

#### Example, from M2's `catalog_product_view.xml`

```xml
<page layout="1column" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="urn:magento:framework:View/Layout/etc/page_configuration.xsd">
    <head>
        <css src="mage/gallery/gallery.css"/>
    </head>
    <update handle="catalog_product_opengraph" />
    <update handle="page_calendar"/>
    <body>
        <attribute name="itemtype" value="http://schema.org/Product" />
        <attribute name="itemscope" value="itemscope"/>
        <referenceBlock name="head.components">
            <block class="Magento\Framework\View\Element\Js\Components" name="checkout_page_head_components" template="Magento_Catalog::js/components.phtml"/>
        </referenceBlock>
        <referenceBlock name="page.main.title">
            <arguments>
                <argument name="css_class" xsi:type="string">product</argument>
                <argument name="add_base_attribute" xsi:type="string">itemprop="name"</argument>
            </arguments>
        </referenceBlock>
    </body>
  </page>
  ```

@[2-4](Inserting a new CSS file into page meta-information)
@[10-12](Adding a new block inside of a block)

---

## Instructions

The individual parts of the layout file that add, remove, modify or move elements.

+++

#### Example

```xml
  <block class="Magento\Catalog\Block\Product\ProductList\Item\AddTo\Compare" name="upsell.product.addto.compare"   template="Magento_Catalog::product/list/addto/compare.phtml"/>
```

+++

### Attributes

Optional information within an instruction. Typically define the changes being made.

+++

#### Example

```xml
  <move element="name.of.an.element" destination="name.of.destination.element" as="new_alias" after="name.of.element.after" before="name.of.element.before"/>
```

Notice `element`, `destination`, `as`, `after` and `before` attributes.

---

## Cool, where do I start?

Two ways of working with layouts:

- **Extend**
- **Override**

---

## 1. Extend layouts

Rather than copy and modify extensive code, you only need to create an extending layout file that contains the changes you want.

+++

To customize the catalog product view at `<Magento_Catalog_module_dir>/view/frontend/layout/catalog_product_view.xml`, make a file with the same name in the same location *in your theme's directory*.

Example: `<theme_dir>/Magento_Catalog/layout/catalog_product_view.xml`

This file should only contain the changes you want to make, while everything else is inherited from the base layout file.

---

### 2. Override layouts

If the amount of customizations is large, you can completely override the necessary layout. 

This means that the new file that you place in the theme will be used instead of the parent theme layout file or base layout file.

+++

```
<theme_dir>
  |__/<Namespace_Module>
    |__/layout
      |__/override
         |__/base
           |--<layout1>.xml
           |--<layout2>.xml
```
*or*
```
<theme_dir>
  |__/<Namespace_Module>
    |__/layout
      |__/override
         |__/theme
            |__/<Parent_Vendor>
               |__/<parent_theme>
                  |--<layout1>.xml
                  |--<layout2>.xml
```


---

## Where are the base files?

Find the directory of the relevant module, and navigate to the `view/frontend/layout/...` directory.

> [Magento has a pretty guide on how to do that.](http://devdocs.magento.com/guides/v2.2/frontend-dev-guide/themes/debug-theme.html)

---

### Why not just modify .phtml files?

- All design updates in a central place
- Leaner
- Update-proof

> Basically, avoid editing .phtml files directly until you absolutely need to.

---

### When to definitely use layouts

- change the position of a block or container using `<move>`
- remove a block or container using the remove attribute
- reorder blocks and container using the before/after attributes
- change the HTML tag or CSS class for an existing container

---

### Then what does Magento do with these .xml files?

The Magento application processes layout files in the following order:

+++

- Collects all layout files from modules. The order is determined by the modules order in the module list from `app/etc/config.php`.

- Determines the sequence of [inherited themes](http://devdocs.magento.com/guides/v2.1/frontend-dev-guide/themes/theme-inherit.html).

+++

- Iterates the sequence of themes from last ancestor to current:
  - Adds all extending theme layout files to the list.
  - Replaces overridden layout files in the list.
  
- Merges all layout files from the list.

---

## What else can I do with this?

+++

### Browser conditional comments

```xml
    <head>
        <css src="css/ie-9.css" ie_condition="IE 9" />
    </head>
```

becomes

```html
<!--[if IE 9]>
<link rel="stylesheet" type="text/css" media="all" href="<your_store_web_address>/pub/static/frontend/<Vendor>/<theme>/en_US/css/ie-9.css" />
<![endif]-->
```

+++

## Removing useless resources

`app/design/frontend/<Vendor>/<theme>/Magento_Theme/
layout/default_head_blocks.xml`

```xml
   <head>
  <!-- Remove local resources -->
        <remove src="css/styles-m.css" />
        <remove src="my-js.js"/>
        <remove src="Magento_Catalog::js/compare.js" />
                
  <!-- Remove external resources -->
        <remove src="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.4/css/bootstrap-theme.min.css"/>
        <remove src="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.4/js/bootstrap.min.js"/>
        <remove src="http://fonts.googleapis.com/css?family=Montserrat" /> 
   </head>
```

+++

## Change a block template to a custom .phtml template

```xml
 <referenceBlock name="page.main.title" 
                 template="%Namespace_Module::new_template.phtml%"/>
```

- `Namespace_Module`: defines the module the template belongs to. For example, Magento_Catalog.
- `new_template.phtml`: the path to the template relatively to the templates directory. It might be `<module_dir>/view/<area>/templates` or `<theme_dir>/<Namespace_Module>/templates`.


+++

## Utilize existing blocks

```xml 
  <referenceContainer name="header-wrapper">
      <container name="desktop-links" as="desktop.links" htmlTag="ul" htmlClass="desktop-nav" after="minicart">
        <block class="Magento\Framework\View\Element\Html\Link" name="orderstatus.link">
          <arguments>
            <argument name="label" xsi:type="string" translate="false">Order Status</argument>
            <argument name="path" xsi:type="string" translate="false">sales/guest/form</argument>
          </arguments>
        </block>
        <block class="Magento\Framework\View\Element\Html\Link" name="account.link">
          <arguments>
            <argument name="label" xsi:type="string" translate="false">My Account</argument>
            <argument name="path" xsi:type="string" translate="false">customer/account</argument>
          </arguments>
        </block>
      </container>
  </referenceContainer>
```

---

## Finally,

## Magento is all about inheritance.

- Everything builds on something else.
- Magento contains 'base' theme and modules.
- Magento has a *lot* of things to re-use.


---

## Further layout research

- [Magento's Frontend Developer Guide](http://devdocs.magento.com/guides/v2.2/frontend-dev-guide/bk-frontend-dev-guide.html)
- [Toptal's blog](https://www.toptal.com/developers/blog/tags/magento2)
- Nothing beats tinkering.

---

![Any questions?](https://media.giphy.com/media/3o6Ztmhk7M5bU9p3kk/giphy.gif)
