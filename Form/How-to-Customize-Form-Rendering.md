#如何自定义表单渲染
Symfony提供了许多方法来自定义表单渲染，在这篇指南里，你会学到如何使用Twig或PHP作为模板来自定义表单中每个可以定制的部分。

##表单渲染的基本
回想一下，表单的标签、错误和HTML部件都可以轻易地通过*Twig*的*form_row*方法或*PHP*的*row*方法来实现：
```Twig
{{ form_row(form.age) }}
```

也可以分别独立地渲染字段的三个部分：
```Twig
<div>
	{{ form_label(form.age) }}
	{{ form_errors(form.age) }}
	{{ form_widget(form.age) }}
</div>
```

在这两种情况下，表单标签、错误和HTML控件是通过使用Symfony标准的一组标记来进行渲染的。例如，以上两个模板的渲染：
```PHP
<div>
	<label for="form_age">Age</label>
	<ul>
		<li>This field is required</li>
	</ul>
	<input type="number" id="form_age" name="form[age]" />
</div>
```

要快速进行原型开发和测试一个表单，你可以仅使用一行代码就渲染整个表单：
```Twig
{{ form_widget(form) }}
```

这个指南的剩余部分将介绍在表单标记的每个部分在几个不同的层次进行修改。关于表单总体渲染的更多信息，请查看[Rendering a Form in a Template](http://symfony.com/doc/current/book/forms.html#form-rendering-template)。

##什么是表单主题？
Symfony使用表单片段 —— 一个小的可以渲染表单一部分的片段 —— 来渲染表单的每一部分 —— 字段标签、错误、输入文本字段、选择标签等。

片段在Twig中被定义为*块*，在PHP中被定义为*模板文件*。

主题是当你渲染一个表单时所使用的一组片段，换句话说，如果你想要自定义渲染表单的一部分，你可以导入一个包含自定义表单片段的主题。

Symfony带有默认的主题（Twig中的[form_div_layout.html.twig](https://github.com/symfony/symfony/blob/master/src/Symfony/Bridge/Twig/Resources/views/Form/form_div_layout.html.twig)和PHP中的*FrameworkBundle:Form*），主题定义了需要渲染的表单的每个片段。

在下一部分你会学到如何通过覆盖一些或所有片段来自定义一个主题。

例如，当一个*integer*类型字段的组件被渲染时，就会生成一个*input number*字段。
```Twig
{{ form_widget(form.age) }}
```

renders:
```
<input type="number" id="form_age" name="form[age]" required="required" value="33" />
```

在内部，Symfony使用*integer_widget*片段来渲染字段，这是因为字段类型是*integer*并且你正在渲染它的组件（同样相对于他的*label*和*errors*）。

在Twig中默认是使用[form_div_layout.html.twig](https://github.com/symfony/symfony/blob/master/src/Symfony/Bridge/Twig/Resources/views/Form/form_div_layout.html.twig)模板的*integer_widget*字段。

在PHP中是使用*FrameworkBundle/Resources/views/Form*文件夹下的*integer_widget.html.php*文件。

默认的不完整的*integer_widget*片段如下：
```Twig
{# form_div_layout.html.twig #}
{% block integer_widget %}
	{% set type = type|default('number') %}
	{% block('form_widget_simple') %}
{% endblock integer_widget %}
```

