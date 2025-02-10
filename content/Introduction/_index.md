+++
title = "Introduction"
type = "chapter"
+++
{{< tabs >}}
{{% tab title="just colored style" style="blue" %}}
The `style` parameter is set to a color style.

This will set the background to a lighter version of the chosen style color as configured in your theme variant.
{{% /tab %}}
{{% tab title="just color" color="blue" %}}
Only the `color` parameter is set.

This will set the background to a lighter version of the chosen CSS color value.
{{% /tab %}}
{{% tab title="default style and color" style="default" color="blue" %}}
The `style` parameter affects how the `color` parameter is applied.

The `default` style will set the background to your `--MAIN-BG-color` as configured for your theme variant resembling the default style but with different color.
{{% /tab %}}
{{% tab title="just severity style" style="info" %}}
The `style` parameter is set to a severity style.

This will set the background to a lighter version of the chosen style color as configured in your theme variant and also affects the chosen icon.
{{% /tab %}}
{{% tab title="severity style and color" style="info" color="blue" %}}
The `style` parameter affects how the `color` parameter is applied.

This will set the background to a lighter version of the chosen CSS color value and also affects the chosen icon.
{{% /tab %}}
{{< /tabs >}}
