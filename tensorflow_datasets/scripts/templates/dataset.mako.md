<%!
import tensorflow_datasets as tfds
from tensorflow_datasets.core.utils.py_utils import get_class_path
from tensorflow_datasets.core.utils.py_utils import get_class_url
%>

## Print URLs
<%def name="display_urls(builder, level)">\
${'#' * level} Urls
% for url in builder.info.urls:
 * [${url}](${url})
%endfor
</%def>

## Print features
<%def name="display_features(builder, level)">\
${'#' * level} Features
```python
${builder.info.features}
```
</%def>

## Print Supervised keys
<%def name="display_supervised_keys(builder, level)">\
%if builder.info.supervised_keys:
${'#' * level} Supervised keys (for `as_supervised=True`)
`${str(builder.info.supervised_keys)}`
%endif
</%def>

## Print the bullet points + features specific to builder with a single version.
<%def name="print_general_info_one_config(builder)">
${display_description(builder)}

* URL: [${builder.info.homepage_url}](${builder.info.homepage_url})
* `DatasetBuilder`: [`${get_class_path(builder)}`](${get_class_url(builder)})
* Version: `v${builder.info.version}`
* Size: `${tfds.units.size_str(builder.info.size_in_bytes)}`

${display_features(builder, level=2)}
${display_stats(builder, level=2)}
${display_urls(builder, level=2)}
${display_supervised_keys(builder, level=2)}
${display_citation(builder.info.citation, level=2)}
</%def>

## Print the configs: list with name/version/size/description + doc for each.
<%def name="print_builder_configs(builder, config_builders)">
<%
len_conf_descs = len(set([c.description for c in  builder.BUILDER_CONFIGS] + [
    builder.info.description]))
%>
%if len_conf_descs == 1 or len_conf_descs > len(builder.BUILDER_CONFIGS):
${display_description(builder)}
%endif

* URL: [${builder.info.homepage_url}](${builder.info.homepage_url})
* `DatasetBuilder`: [`${get_class_path(builder)}`](${get_class_url(builder)})

`${builder.name}` is configured with `${get_class_path(builder.builder_config)}` and has
the following configurations predefined (defaults to the first one):

%for config, config_builder in zip(builder.BUILDER_CONFIGS, config_builders):
<%
  size = tfds.units.size_str(config_builder.info.size_in_bytes)
%>
* `${config.name}` (`v${config.version}`) (`Size: ${size}`): ${config.description}
%endfor

%for config, config_builder in zip(builder.BUILDER_CONFIGS, config_builders):
${'##'} `${builder.name}/${config.name}`
${config.description}
${display_stats(config_builder, level=3)}
${display_features(config_builder, level=3)}
${display_urls(config_builder, level=3)}
${display_supervised_keys(config_builder, level=3)}
%endfor
${display_citation(config_builder.info.citation, level=2)}
</%def>

## Display the description of a builder.
<%def name="display_description(builder)">\
${builder.info.description}
</%def>

## Display stats for a split.
<%def name="display_stats(builder, level)">\
<%
  splits = builder.info.splits
  size_name = [(split_info.num_examples, split_name)
               for (split_name, split_info) in splits.items()]
%>\
${'#' * level} Statistics
%if builder.info.splits.total_num_examples:
Split  | Examples
:----- | ---:
ALL    | ${"{:,}".format(splits.total_num_examples)}
%for split_size, split_name in sorted(size_name, key=lambda x:(-x[0], x[1])):
${split_name.upper()} | ${"{:,}".format(split_size)}
%endfor
%else:
None computed
%endif
</%def>

## Display a citation.
<%def name="display_citation(citation, level)">\
%if citation:
${'#' * level} Citation
```
${citation}
```
%endif
</%def>

# `${builder.name}`

%if builder.builder_config:
${print_builder_configs(builder, config_builders)}
%else:
${print_general_info_one_config(builder)}
%endif
---
