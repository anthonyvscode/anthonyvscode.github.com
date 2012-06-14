---
title: "MVC 3 RequiredIf validator for multiple values"
date: 2011-07-14
layout: post
---
Recently I was required to create a form that included the dependencies of "If zzz field equals xx, then yyy field is required" hierarchical type structure to my viewmodel, and came across <a href="http://blogs.msdn.com/b/simonince/archive/2011/02/04/conditional-validation-in-asp-net-mvc-3.aspx">this awesome post</a> by Simon Ince describing exactly how to do it for a 1:1 relationship using DataAttributes...

Problem for me: I needed the field to be Required if the selected value in my dropdownlist was either 1, 2 or 3. It was not required if the value was 0.

<h4><em>Uh ohhhhhhh......</em></h4>

well, I changed his control a bit and it was surprisingly easy to do (cheers, microsoft). Here is my code! (note: if you use my code instead of Simon's, its the exact same syntax to call it so you can switch it out with this one with no repercussions, this one just accepts an unlimited number of items to compare against if need be not just a single one).

<pre class="prettyprint">
using System;
using System.Collections.Generic;
using System.Linq;
using System.Web;
using System.ComponentModel.DataAnnotations;
using System.Web.Mvc;
using System.Collections;
using System.Text;

namespace MyProject.Validation.ValidationAttributes
{
    public class RequiredIfAttribute : ValidationAttribute, IClientValidatable
    {
        private RequiredAttribute _innerAttribute = new RequiredAttribute();

        private string _dependentProperty;
        private object[] _targetValue;

        public RequiredIfAttribute(string dependentProperty, params object[] targetValue)
        {
            this._dependentProperty = dependentProperty;
            this._targetValue = targetValue;
        }

        protected override ValidationResult IsValid(object value, ValidationContext validationContext)
        {
            // get a reference to the property this validation depends upon
            var containerType = validationContext.ObjectInstance.GetType();
            var field = containerType.GetProperty(this._dependentProperty);

            if (field != null)
            {
                // get the value of the dependent property
                var dependentvalue = field.GetValue(validationContext.ObjectInstance, null);

                foreach (var obj in _targetValue)
                {
                    // compare the value against the target value
                    if ((dependentvalue == null &amp;&amp; this._targetValue == null) ||
                        (dependentvalue != null &amp;&amp; dependentvalue.Equals(obj)))
                    {
                        // match =&gt; means we should try validating this field
                        if (!_innerAttribute.IsValid(value))
                            // validation failed - return an error
                            return new ValidationResult(this.ErrorMessage, new[] { validationContext.MemberName });
                    }
                }
            }

            return ValidationResult.Success;
        }

        public IEnumerable&lt;ModelClientValidationRule&gt; GetClientValidationRules(ModelMetadata metadata, ControllerContext context)
        {
            var rule = new ModelClientValidationRule()
            {
                ErrorMessage = FormatErrorMessage(metadata.GetDisplayName()),
                ValidationType = &quot;requiredif&quot;,
            };

            string depProp = BuildDependentPropertyId(metadata, context as ViewContext);

            // find the value on the control we depend on;
            // if it's a bool, format it javascript style 
            // (the default is True or False!)

            StringBuilder sb = new StringBuilder();

            foreach (var obj in this._targetValue)
            {
                string targetValue = (obj ?? &quot;&quot;).ToString();

                if (obj.GetType() == typeof(bool))
                    targetValue = targetValue.ToLower();

                sb.AppendFormat(&quot;|{0}&quot;, targetValue);
            }

            rule.ValidationParameters.Add(&quot;dependentproperty&quot;, depProp);
            rule.ValidationParameters.Add(&quot;targetvalue&quot;, sb.ToString().TrimStart('|'));

            yield return rule;
        }

        private string BuildDependentPropertyId(ModelMetadata metadata, ViewContext viewContext)
        {
            // build the ID of the property
            string depProp = viewContext.ViewData.TemplateInfo.GetFullHtmlFieldId(this._dependentProperty);
            // unfortunately this will have the name of the current field appended to the beginning,
            // because the TemplateInfo's context has had this fieldname appended to it. Instead, we
            // want to get the context as though it was one level higher (i.e. outside the current property,
            // which is the containing object (our Person), and hence the same level as the dependent property.
            var thisField = metadata.PropertyName + &quot;_&quot;;
            if (depProp.StartsWith(thisField))
                // strip it off again
                depProp = depProp.Substring(thisField.Length);
            return depProp;
        }
    }
}

</pre>

<h3>PS, It does client validation too.</h3>

<pre class="prettyprint">
$.validator.addMethod('requiredif',
    function (value, element, parameters) {
        var id = '#' + parameters['dependentproperty'];
        // get the target value (as a string, 
        // as that's what actual value will be)
        var targetvalue = parameters['targetvalue'];
        targetvalue = (targetvalue == null ? '' : targetvalue).toString();

        var targetvaluearray = targetvalue.split('|');

        for (var i = 0; i &lt; targetvaluearray.length; i++) {

            // get the actual value of the target control
            // note - this probably needs to cater for more 
            // control types, e.g. radios
            var control = $(id);
            var controltype = control.attr('type');
            var actualvalue =
            controltype === 'checkbox' ?
            control.attr('checked') ? &quot;true&quot; : &quot;false&quot; :
            control.val();

            // if the condition is true, reuse the existing 
            // required field validator functionality
            if (targetvaluearray[i] === actualvalue) {
                return $.validator.methods.required.call(this, value, element, parameters);
            }
        }

        return true;
    }
);

$.validator.unobtrusive.adapters.add(
    'requiredif',
    ['dependentproperty', 'targetvalue'],
    function (options) {
        options.rules['requiredif'] = {
            dependentproperty: options.params['dependentproperty'],
            targetvalue: options.params['targetvalue']
        };
        options.messages['requiredif'] = options.message;
    });
</pre>

<h3>So, how do I use this and why is it useful?</h3>


For my example, I created an Enum to encapsulate my values to compare against. makes it alot neater and easier to keep track of when you have alot of requiredif validators on various fields.

<pre class="prettyprint">
public enum ApplicationStatus
{
    Pending = 0,
    Submitted = 1,
    Disapproved = 2,
    Approved = 3
}

</pre>

And In my viewmodel, I wanted my ApplicationSubmitted field required only if the application has actually been submitted (aka, past the "pending" stage.)

<pre class="prettyprint">
public SelectList ApplicationStatus{ get; set; }

[Required]
public int SelectedApplicationStatus { get; set; }

[RequiredIf(&quot;SelectedApplicationStatus&quot;,
    (int)Enums.ApplicationStatus.Submitted,
    (int)Enums.ApplicationStatus.Approved,
    (int)Enums.ApplicationStatus.Disapproved, ErrorMessage=&quot;The Application Submission Date is required&quot;)]
public DateTime? ApplicationSubmitted { get; set; } 

</pre>

And with a simple line to display your validation message like so:

<pre class="prettyprint">
@Html.ValidationMessageFor(m =&gt; m.ApplicationSubmitted)
</pre>

You're right to go.

<h2>BALLIN'</h2>

Let us know what you think