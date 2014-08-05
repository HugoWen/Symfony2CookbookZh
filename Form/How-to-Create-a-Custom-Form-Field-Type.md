#如何创建自定义表单字段类型
Symfony带有一系列建立表单的的核心字段类型，然而，有时候你可能存在特殊的需要来创建自定义的表单字段类型。这里的例子假设你需要基于现有的选择字段来定义一个新的字段来保存一个人的性别。本节介绍字段是如何定义的，如何自定义显示，以及最终如何注册并在程序中使用。

##定义字段类型
为了创建自定义的字段类型，首先你应该创建一个表示字段的类，在本例子的情况下，用于保存字段类型的类叫做*GenderType*，类文件保存在默认的表单字段位置*<BundleName>\Form\Type*。确保字段类是继承*AbstractType*类的。
```php
// src/Acme/DemoBundle/Form/Type/GenderType.php
namespace Acme\DemoBundle\Form\Type;

use Symfony\Component\Form\AbstractType;
use Symfony\Component\OptionsResolver\OptionsResolverInterface;

class GenderType extends AbstractType
{
    public function setDefaultOptions(OptionsResolverInterface $resolver)
    {
        $resolver->setDefaults(array(
            'choices' => array(
                'm' => 'Male',
                'f' => 'Female',
            )
        ));
    }

    public function getParent()
    {
        return 'choice';
    }

    public function getName()
    {
        return 'gender';
    }
}
```

> 这个类文件的位置不是十分重要的——放在*\Form\Type*目录下只是一个惯例。

在这里，*getParent*方法的返回值表示你在扩展*choice*字段类型，这就意味着，默认情况下，你在继承该字段类型的所有逻辑和渲染。这一部分的逻辑是定义在*ChoiceType*类文件里，有三个方法是尤为重要的：
- *buildForm()* —— 每个字段类型都有一个*buildForm()*方法，在这个方法中你配置和构建任意字段。注意这也是你设置表单时使用的同样的方法，它也同样在这里工作。
- *buildView()* —— 这个方法是用来设置在你渲染模板中变量时所需要的任意扩展变量。例如，在*ChoiceType*中，多变量被设置并在模板中使用来设置（或没有）选定字段的多属性。参考[给字段创建模板](http://symfony.com/doc/current/cookbook/form/create_custom_field_type.html#creating-a-template-for-the-field)。
- *setDefaultOptions()* —— 这个方法是为在*buildForm()*和*buildView()*方法中定义使用的表单类型的选项。有许多字段的普通选项（参考[表单字段类型](http://symfony.com/doc/current/reference/forms/types/form.html)），但在这里你也可以创建你需要的任何其他选项。

> 如果你在创建一个由许多字段构成的字段，那么确保设置你的*"父"*类型作为*表单*或其他的继承自*表单*。同样，如果你需要从你的父类型中更改任意子类型的*"视图"*，那么就使用*finishView()*方法。

*getName()*方法在应用中返回一个唯一的标识符，这将在许多地方使用，比如当自定义如何渲染表单类型的时候。

这个字段的目标是扩展选项类型以使性别选择可用，这是通过固定选项到可能性别的列表中来实现的。

##给字段创建一个模板
每个字段类型都由一个模板片段渲染呈现，这部分是在你的*getName()*方法中定义的。要了解更多信息，请查看[什么是表单主题](http://symfony.com/doc/current/cookbook/form/form_customization.html#cookbook-form-customization-form-themes)。

在这个例子中，由于父字段是*choice*，因为自定义字段类型会自动地渲染一个*choice*类型模板，所以你不需要做任何工作。但是，由于这个例子是假设当你的字段是*可扩展的*（比如：radio button或checkboxes来代替选择字段），你想要总是以一种*ul*元素来渲染它。在你的表单主题模板中（详细信息参考上面的链接内容），创建一个*gender_widget*模块来处理这部分：
```twig
{# src/Acme/DemoBundle/Resources/views/Form/fields.html.twig #}
{% block gender_widget %}
    {% spaceless %}
        {% if expanded %}
            <ul {{ block('widget_container_attributes') }}>
            {% for child in form %}
                <li>
                    {{ form_widget(child) }}
                    {{ form_label(child) }}
                </li>
            {% endfor %}
            </ul>
        {% else %}
            {# just let the choice widget render the select tag #}
            {{ block('choice_widget') }}
        {% endif %}
    {% endspaceless %}
{% endblock %}
```

> 确保使用正确的部件前缀，在这个例子里，根据*genName*方法返回的值，名字应当是*gender_widget*。并且主配置文件应当指向自定义的表单模板以便在渲染所有表单时都能够使用。
```yaml
# app/config/config.yml
twig:
    form:
        resources:
            - 'AcmeDemoBundle:Form:fields.html.twig'
```

##使用字段类型
现在可以立刻使用自定义字段类型，可以在你的表单中很简单地创建一个新的类型接口：
```php
// src/Acme/DemoBundle/Form/Type/AuthorType.php
namespace Acme\DemoBundle\Form\Type;

use Symfony\Component\Form\AbstractType;
use Symfony\Component\Form\FormBuilderInterface;

class AuthorType extends AbstractType
{
    public function buildForm(FormBuilderInterface $builder, array $options)
    {
        $builder->add('gender_code', new GenderType(), array(
            'empty_value' => 'Choose a gender',
        ));
    }
}
```

但是这种*GenderType()*比较简单，当gender的代码存储在配置文件中或在数据库中回事怎样呢？下一部分解释了如何使用更加复杂的字段类型来解决这一问题。

##创建你的字段类型作为一个服务
到目前为止，这个实例已经假设你有一个非常简单的自定义字段类型。但是如果你需要访问配置、数据库连接或者一些其他的服务，那么你要注册自定义类型的服务。例如，假设你在配置文件中存储性别参数：
```yaml
# app/config/config.yml
parameters:
    genders:
        m: Male
        f: Female
```
为了使用这个参数，需要定义你的自定义字段类型作为一个服务，注入*性别*参数值作为它将要创建的*构造*方法的参数：
```yaml
# src/Acme/DemoBundle/Resources/config/services.yml
services:
    acme_demo.form.type.gender:
        class: Acme\DemoBundle\Form\Type\GenderType
        arguments:
            - "%genders%"
        tags:
            - { name: form.type, alias: gender }
```

> 确保服务文件是被引入的。详细参考[使用imports引入配置文件](http://symfony.com/doc/current/book/service_container.html#service-container-imports-directive)。

确保标签的*alias*属性对应于早先定义的*getName*方法返回的值。你会在一瞬间就发现这个的重要性当你使用自定义字段类型时。但是首先，给*GenderType*添加一个*构造*方法，这个方法用来接收gender的配置：
```php
// src/Acme/DemoBundle/Form/Type/GenderType.php
namespace Acme\DemoBundle\Form\Type;

use Symfony\Component\OptionsResolver\OptionsResolverInterface;

// ...

// ...
class GenderType extends AbstractType
{
    private $genderChoices;

    public function __construct(array $genderChoices)
    {
        $this->genderChoices = $genderChoices;
    }

    public function setDefaultOptions(OptionsResolverInterface $resolver)
    {
        $resolver->setDefaults(array(
            'choices' => $this->genderChoices,
        ));
    }

    // ...
}
```

好的，*GenderType*方法现在得益于配置的参数并已经注册成为了一个服务。更多的，由于你在配置文件中使用了*form.type*的alias别名，现在使用此字段会更加简单：
```php
// src/Acme/DemoBundle/Form/Type/AuthorType.php
namespace Acme\DemoBundle\Form\Type;

use Symfony\Component\Form\FormBuilderInterface;

// ...

class AuthorType extends AbstractType
{
    public function buildForm(FormBuilderInterface $builder, array $options)
    {
        $builder->add('gender_code', 'gender', array(
            'empty_value' => 'Choose a gender',
        ));
    }
}
```

注意到取代实例化一个新的实例，你可以仅通过alias别名在服务配置中的使用来引用*gender*。

翻译自：[http://symfony.com/doc/current/cookbook/form/create_custom_field_type.html#creating-a-template-for-the-field](http://symfony.com/doc/current/cookbook/form/create_custom_field_type.html#creating-a-template-for-the-field)