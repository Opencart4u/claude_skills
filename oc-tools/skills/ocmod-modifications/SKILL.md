---
name: ocmod-modifications
description: >
  Write and troubleshoot OpenCart OCMOD modification files (install.xml / *.ocmod.xml). Use this
  skill whenever creating, editing, or debugging OCMOD modifications for OpenCart — injecting code
  with search/add operations, choosing before/after/replace positions, disambiguating repeated
  lines with index, globbing files with wildcards/braces, guarding against double application with
  ignoreif, using regex anchors as a last resort, handling Greek/UTF-8 content in CDATA, or
  diagnosing why a modification refresh did not apply.
---

# OCMOD worked examples & troubleshooting

Every example obeys the core rule: **one line per `<search>`**.

## 1. Inject code AFTER a line (preferred for additions)

Add a value to the home-page controller data without replacing anything.

```xml
<?xml version="1.0" encoding="utf-8"?>
<modification>
    <name>Home Extra Data</name>
    <version>1.0</version>
    <author>Digital4u</author>
    <link>https://digital4u.gr</link>
    <file path="catalog/controller/common/home.php">
        <operation>
            <search><![CDATA[$data['column_left'] = $this->load->controller('common/column_left');]]></search>
            <add position="after"><![CDATA[
        $data['my_flag'] = true;
            ]]></add>
        </operation>
    </file>
</modification>
```

## 2. Inject BEFORE a line

```xml
<operation>
    <search><![CDATA[return $this->load->view('common/home', $data);]]></search>
    <add position="before"><![CDATA[
        $data['my_flag'] = true;
    ]]></add>
</operation>
```

## 3. Replace a single line

Only the matched line is swapped — `<add>` carries the full replacement.

```xml
<operation>
    <search><![CDATA[$this->response->setOutput($this->load->view('common/home', $data));]]></search>
    <add position="replace"><![CDATA[
        $this->response->setOutput($this->load->controller('common/maintenance'));
    ]]></add>
</operation>
```

## 4. Disambiguate a repeated line with `index`

If `$json = array();` appears several times and you want the 2nd one:

```xml
<operation>
    <search index="2"><![CDATA[$json = array();]]></search>
    <add position="after"><![CDATA[
        $json['custom'] = true;
    ]]></add>
</operation>
```

## 5. Span several lines WITHOUT a multi-line search

Wrong (brittle — multi-line search):

```xml
<!-- DON'T: whitespace/version differences break this -->
<search><![CDATA[
    $data['products'] = array();
    foreach ($results as $result) {
]]></search>
```

Right (anchor on one line, use `offset` to reach a nearby line):

```xml
<operation>
    <search><![CDATA[foreach ($results as $result) {]]></search>
    <add position="after"><![CDATA[
            // runs first inside the loop body
    ]]></add>
</operation>
```

Or apply two separate single-line operations, each anchored on its own unique line.

## 6. One operation across many files (glob / braces)

```xml
<file path="catalog/view/theme/*/template/common/header.twig">
    <operation error="skip">
        <search><![CDATA[<ul class="nav navbar-nav">]]></search>
        <add position="after"><![CDATA[
            <li><a href="{{ catalog }}index.php?route=product/special">Offers</a></li>
        ]]></add>
    </operation>
</file>
```

`error="skip"` is important here: not every theme folder will contain that exact line, and without
`skip` a single missing match can break the whole refresh.

Braces target several specific files at once:

```xml
<file path="system/{engine,library}/{action,loader}*.php">
```

## 7. Guard against double application with `ignoreif`

```xml
<operation ignoreif="my_already_added_marker">
    <search><![CDATA[$data['column_left'] = $this->load->controller('common/column_left');]]></search>
    <add position="after"><![CDATA[
        $data['my_already_added_marker'] = true;
    ]]></add>
</operation>
```

## 8. Regex (last resort)

Use only when no stable literal single-line anchor exists. `position`/`trim`/`offset` are ignored.

```xml
<operation>
    <search regex="true" limit="1"><![CDATA[~\$data\['heading_title'\]\s*=\s*[^;]+;~]]></search>
    <add><![CDATA[$data['heading_title'] = 'Custom Title';]]></add>
</operation>
```

## Greek / UTF-8

Keep Greek text literally inside CDATA with `encoding="utf-8"` on the XML declaration — do not HTML-
entity-escape it. CDATA is opaque, so quotes and special characters inside are safe.

```xml
<add position="after"><![CDATA[
    $data['promo_text'] = 'Δωρεάν αποστολή άνω των 50€';
]]></add>
```

Save the `install.xml` file itself as UTF-8 without BOM.

---

## Troubleshooting

**"Modification refresh shows no error but nothing changed."**
The search didn't match. Almost always the `<search>` was multi-line or carried indentation that
isn't in the source. Reduce it to one unique line and ensure `trim="true"` behaviour.

**"It worked on one store but not another / broke after an OpenCart update."**
The anchor line changed or differs by theme. Re-pick a more stable single line, or switch a
`replace` to `before`/`after` so you depend on less of the surrounding code.

**"Operation applied in the wrong place."**
The anchor line repeats in the file. Add `index="N"` instead of widening the search.

**"Refresh aborts / logs an error for a wildcard path."**
One of the globbed files lacks the line. Add `error="skip"` to the operation.

**"Mod stops working when another mod also edits the same file."**
OCMOD applies mods sequentially to the same virtual copy. If an earlier mod already rewrote your
anchor line, yours won't match. Anchor on a line the other mod leaves untouched, or coordinate
ordering. Use the modification **Log** tab in admin to see what each mod did.

**"Uploaded file rejected by the installer."**
The archive must be named `*.ocmod.zip` (or the standalone XML `*.ocmod.xml`), and every `path`
must start with `admin`, `catalog`, or `system`.
