---
title: Patch Kolla via Template
author: Date Huang
date: 2018-10-18T12:31:32+00:00
url: /2018/10/patch-kolla-via-template/
categories:
  - openstack

---
Kolla 多少還是有一些不合需求的地方，我們可以透過 Kolla Template 的方式修改 Dockerfile ，不用直接硬修改 Kolla 本身，而是透過 Kolla 提供的 Plugin 安裝接口修正。

修改的同時可能需要一些 Jinja (Template Engine) 的一些知識，可以參考 [Jinja 官網][1]的文件，也需要 Dockerfile 的撰寫知識。

<!--more-->

<div id="ez-toc-container" class="ez-toc-v2_0_17 counter-hierarchy counter-decimal ez-toc-grey">
  <div class="ez-toc-title-container">
    <p class="ez-toc-title">
      Table of Contents
    </p>
    
    <span class="ez-toc-title-toggle"><a class="ez-toc-pull-right ez-toc-btn ez-toc-btn-xs ez-toc-btn-default ez-toc-toggle" style="display: none;"><i class="ez-toc-glyphicon ez-toc-icon-toggle"></i></a></span>
  </div><nav>
  
  <ul class="ez-toc-list ez-toc-list-level-1">
    <li class="ez-toc-page-1 ez-toc-heading-level-2">
      <a class="ez-toc-link ez-toc-heading-1" href="https://blog.kojuro.date/2018/10/patch-kolla-via-template/#%E7%B0%A1%E5%96%AE%E6%95%99%E5%AD%B8" title="簡單教學">簡單教學</a>
    </li>
    <li class="ez-toc-page-1 ez-toc-heading-level-2">
      <a class="ez-toc-link ez-toc-heading-2" href="https://blog.kojuro.date/2018/10/patch-kolla-via-template/#%E6%B3%9B%E7%94%A8_Template" title="泛用 Template">泛用 Template</a>
    </li>
    <li class="ez-toc-page-1 ez-toc-heading-level-2">
      <a class="ez-toc-link ez-toc-heading-3" href="https://blog.kojuro.date/2018/10/patch-kolla-via-template/#Reference" title="Reference">Reference</a>
    </li>
  </ul></nav>
</div>

## <span class="ez-toc-section" id="%E7%B0%A1%E5%96%AE%E6%95%99%E5%AD%B8"></span>簡單教學<span class="ez-toc-section-end"></span>

可以修改的地方非常多，我這邊就講解最簡單的 Patch 項目。

一般而言我們上 Patch 都會在最後的最後，才把我們修正的部分加進去，避免中間操作，又把我們修正的部分還原。Kolla Dockerfile 中提供了 `<service image>-footer` 的區塊，供我們插入 Plugin 或是其他額外的檔案設定。

就例如說前文，我們要修正 QEMU 的軟體包，那我們可以在 `template-override.j2` 這個檔案中修正 Nova-Libvirt 。

<pre class="wp-block-code"><code># in template-override.j2
{% extends parent_template %}

{% block nova_libvirt_footer %}
ADD \
    http://&lt;some ip>/qemu-utils_2.11+dfsg-1ubuntu7.6_arm64.deb \
    /patch/
RUN dpkg -i /patch/*.deb
{% endblock %}</code></pre>

Kolla 建置 Image 的時候新增以下參數

<pre class="wp-block-code"><code>tools/build.py --template-override template-override.j2</code></pre>

就可以在建置最後，直接將我們的修改好的 QEMU 直接加入其中

## <span class="ez-toc-section" id="%E6%B3%9B%E7%94%A8_Template"></span>泛用 Template<span class="ez-toc-section-end"></span>

當然我們也可以直接撰寫一個泛用的 template，我的情境需要用到 x86_64 以及 arm64 兩種環境，所以我可以改成泛用的 template 以供使用。

<pre class="wp-block-code"><code>{% extends parent_template %}

{% set custom_patch_url = "http://mgmt.kojuro.date:8000/" %}
{% set custom_patch_path = "/patch/" %}
{% set custom_patch_version = "2.11+dfsg-1ubuntu7.6" %}
{% set custom_patch_arch = "amd64" %}
{% if base_arch == "aarch64" %}
    {% set custom_patch_arch = "arm64" %}
{% endif %}

{% block cinder_base_footer %}
# qemu-utils qemu-block-extra
ADD \
    {{ custom_patch_url }}/qemu-utils_{{ custom_patch_version}}_{{ custom_patch_arch }}.deb \
    {{ custom_patch_url }}/qemu-block-extra_{{ custom_patch_version}}_{{ custom_patch_arch }}.deb \
    {{ custom_patch_path }}/ 
RUN dpkg -i {{ custom_patch_path }}/*.deb
{% endblock %}

{% block nova_libvirt_footer %}
# qemu qemu-block-extra
# qemu-system qemu-user qemu-utils 
# qemu-system-arm qemu-system-mips qemu-system-ppc qemu-system-sparc qemu-system-x86 qemu-system-s390x qemu-system-misc
# qemu-system-common
ADD \
    {{ custom_patch_url }}/qemu-utils_{{ custom_patch_version}}_{{ custom_patch_arch }}.deb \
    {{ custom_patch_url }}/qemu-system-arm_{{ custom_patch_version}}_{{ custom_patch_arch }}.deb \
    {{ custom_patch_url }}/qemu-system-mips_{{ custom_patch_version}}_{{ custom_patch_arch }}.deb \
    {{ custom_patch_url }}/qemu-system-ppc_{{ custom_patch_version}}_{{ custom_patch_arch }}.deb \
    {{ custom_patch_url }}/qemu-system-sparc_{{ custom_patch_version}}_{{ custom_patch_arch }}.deb \
    {{ custom_patch_url }}/qemu-system-x86_{{ custom_patch_version}}_{{ custom_patch_arch }}.deb \
    {{ custom_patch_url }}/qemu-system-s390x_{{ custom_patch_version}}_{{ custom_patch_arch }}.deb \
    {{ custom_patch_url }}/qemu-system-misc_{{ custom_patch_version}}_{{ custom_patch_arch }}.deb \
    {{ custom_patch_url }}/qemu-system-common_{{ custom_patch_version}}_{{ custom_patch_arch }}.deb \
    {{ custom_patch_url }}/qemu-user_{{ custom_patch_version}}_{{ custom_patch_arch }}.deb \
    {{ custom_patch_url }}/qemu_{{ custom_patch_version}}_{{ custom_patch_arch }}.deb \
    {{ custom_patch_url }}/qemu-block-extra_{{ custom_patch_version}}_{{ custom_patch_arch }}.deb \ 
    {{ custom_patch_path }}/ 
RUN dpkg -i {{ custom_patch_path }}/*.deb
{% endblock %}

{% block nova_compute_footer %}
# qemu-utils qemu-block-extra
ADD \
    {{ custom_patch_url }}/qemu-utils_{{ custom_patch_version}}_{{ custom_patch_arch }}.deb \
    {{ custom_patch_url }}/qemu-block-extra_{{ custom_patch_version}}_{{ custom_patch_arch }}.deb \
    {{ custom_patch_path }}/ 
RUN dpkg -i {{ custom_patch_path }}/*.deb
{% endblock %}

{% block glance_api_footer %}
# qemu-utils qemu-block-extra
ADD \
    {{ custom_patch_url }}/qemu-utils_{{ custom_patch_version}}_{{ custom_patch_arch }}.deb \
    {{ custom_patch_url }}/qemu-block-extra_{{ custom_patch_version}}_{{ custom_patch_arch }}.deb \
    {{ custom_patch_path }}/ 
RUN dpkg -i {{ custom_patch_path }}/*.deb
{% endblock %}
</code></pre>

## <span class="ez-toc-section" id="Reference"></span>Reference<span class="ez-toc-section-end"></span>

[OpenStack Docs Building Container Images Dockerfile Customisation][2]

 [1]: http://jinja.pocoo.org/docs/2.10/
 [2]: https://docs.openstack.org/kolla/latest/admin/image-building.html#dockerfile-customisation
