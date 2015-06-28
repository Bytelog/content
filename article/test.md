Title of the Article
====================

Lorem ipsum dolor sit amet, consectetur adipiscing elit. Donec a diam lectus. *Sed sit amet ipsum mauris. Maecenas congue ligula ac quam viverra nec consectetur ante hendrerit*. Donec et mollis dolor. Praesent et diam eget libero egestas mattis sit amet vitae augue. Nam tincidunt congue enim, ut porta lorem lacinia consectetur. 


This is an Article Section Heading
----------------------------------

Donec ut libero sed arcu vehicula ultricies a non tortor. Lorem ipsum dolor sit amet, consectetur adipiscing elit. Aenean ut gravida lorem. Ut turpis felis, pulvinar a semper sed, adipiscing id dolor. Pellentesque auctor nisi id magna consequat sagittis. **Curabitur** dapibus enim sit amet elit pharetra tincidunt feugiat nisl imperdiet. Ut convallis libero in urna ultrices accumsan. Donec sed odio eros. Donec viverra mi quis quam pulvinar at malesuada arcu rhoncus. Cum sociis natoque penatibus et magnis dis parturient montes, nascetur ridiculus mus. In rutrum accumsan ultricies. Mauris vitae nisi at sem facilisis semper ac in est.

This is an Article Section Heading
----------------------------------

Vivamus fermentum semper porta. Nunc diam velit, adipiscing ut tristique vitae, sagittis vel odio. Maecenas convallis ullamcorper ultricies. Curabitur ornare, ligula semper consectetur sagittis, nisi diam iaculis velit, id fringilla sem nunc vel mi.

- Here is an item
- And another
- I like lists
- Testing lists!

And, some code for those interested:
```c
#include "lwan.h"
#include "lwan-template.h"
#include "lwan-serve-files.h"
#include "content.h"
#include <stdlib.h>
#include <time.h>

lwan_http_status_t get_article(lwan_request_t *req,
    lwan_response_t *res, void *data)
{
    strbuf_t *filename = strbuf_new();
    strbuf_printf(filename, "%s.md", req->url.value);

    data_t *d = (data_t *) data;

    tpl_article_data_t tpl_data = {
        .title = "Dylan Allbee",
        .description = "Temporary desc",
        .content = parse_markdown_file(req, "content/articles/", strbuf_get_buffer(filename)),
        .year = 2015
    };

    lwan_tpl_apply_with_buffer(d->templates, res->buffer, &tpl_data);
    free(tpl_data.content);
    strbuf_free(filename);

    res->mime_type = "text/html";
    return HTTP_OK;
}

int main(void)
{
    lwan_t l;
    lwan_init(&l);

    const lwan_var_descriptor_t desc_article[] = DESCRIPTOR_ARTICLE;

    data_t data = {
        .templates = lwan_tpl_compile_file("static/templates/article.tpl", desc_article)
    };

    const lwan_url_map_t url_map[] = {
        { .prefix = "/articles", .handler = get_article, .data = &data },
        { .prefix = "/article/", .handler = get_article, .data = &data },
        { .prefix = "/projects", .handler = get_article, .data = &data },
        //{ .prefix = "/project/", .handler = get_article, .data = &data },
        //{ .prefix = "/privacy", .handler = get_article, .data = &data },
        { .prefix = "/", SERVE_FILES("./static/public") },
        { .prefix = NULL },
    };

    lwan_set_url_map(&l, url_map);
    lwan_main_loop(&l);
    lwan_shutdown(&l);
    lwan_tpl_free(data.templates);
    return 0;
}
```


Some Reference Links
--------------------
[google](google.com)  
[yahoo](yahoo.com)  
[duckduckgo](duckduckgo.com)  

