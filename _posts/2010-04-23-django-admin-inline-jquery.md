---
layout: post
title: (A better) Add and Remove Inline Objects from the Django Admin interface using jQuery
date: 2010-04-23 17:30
disqus_identifier: 543916687
---

In my ongoing build out of the platform that will replace our current off the shelf customer and infrastructure management software I've been hacking on the Django Admin interface in terms of usability for our other employees who need a more intuitive user experience.

On the chopping block today was Inline objects and the ability to dynamically add and remove them.

If you do some search on this topic you will most likely end up at [Arne Brodowski's blog](http://www.arnebrodowski.de/blog/507-Add-and-remove-Django-Admin-Inlines-with-JavaScript.html). While it works, my biggest problem with this is that it duplicated the entire table when adding new objects. Ugly...

I updated the code so that it operates on the existing table simply adding & removing table rows for objects. It also toggles the row classes when adding new rows so that the aesthetics is maintained.

Here's the updated code.

{% highlight javascript %}
$(function() {
        // find all of the delete checkboxes
        $('.inline-group .inline-related .delete input').each(function(i,e) {
                // bind to their change event
                $(e).bind('change', function() {
                        if ((this.checked) && ($(this).parent().parent().is(":visible"))) {
                                // if the checkbox is checked and the tr that contains it is
                                // visible, then we hide it
                                $(this).parent().parent().hide();
                        }
                });
        });
});

function add_inline_form(name) {
        // get the tr of the first form item, and then count its siblings
        var first = $('#id_'+name+'-0-id').parent().parent();
        var count = $(first).siblings().length;

        // if we don't have any siblings we're also the last element
        // this allows the code to work when there's no existing inline objects
        if (count == 0) {
                var last = $(first);
        } else {
                var last = $(first).siblings().last();
        }

        // copy the last element
        var copy = $(last).clone(true);
        $(copy).find('[id^="id_'+name+'-'+count+'-"]').each(function(index) {
                // go through each item in the cloned copy that has an id matching
                // and increment the id and name by one
                $(this).attr('id', $(this).attr('id').replace("id_"+name+"-"+count+"-", "id_"+name+"-"+(count+1)+"-"));
                $(this).attr('name', $(this).attr('name').replace(name+"-"+count+"-", name+"-"+(count+1)+"-"));
        });

        // toggle the row class
        $(copy).toggleClass('row1 row2');

        // add the new copy after the last item
        $(last).after(copy);

        // increase the total number of forms so Django will process them
        // account for the original form we cloned and the new one
        $('input#id_'+name+'-TOTAL_FORMS').val(count+2);
        return false;
}
{% endhighlight %}

To use this, or rather the way I use this is, copy the `admin/edit_inline/tabular.html` file from the Django distribution to your local templates directory, then at the bottom of the file copy and paste the above code in between a `<script type="text/javascript"> ... </script>` tag. Make sure you set extra to 1 in the Inline Model Admin and you're good to go.
