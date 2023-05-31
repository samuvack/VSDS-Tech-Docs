
---
sort: 20
---

# RDF syntax

Sisters: <br />
{% assign resultset = page.rdf | rdf_property: '<http://www.ifi.uio.no/INF3580/family#hasSister>', nil, true %}
<ul>
{% for result in resultset %}
    <li>{{ result }}</li>
{% endfor %}
</ul>


Book titles: <br />
{% assign resultset = page.rdf | rdf_property: '<http://xmlns.com/foaf/0.1/currentProject>','de' %}
<ul>
{% for result in resultset %}
    <li>{{ result }}</li>
{% endfor %}
</ul>

{% assign list = page.rdf | rdf_collection: '<http://example.org/directList>' %}
<ol>
{% for item in list %}
<li>{{ item }}</li>
{% endfor %}
</ol>

{% assign container = page.rdf | rdf_property: '<http://example.org/hasContainer>' | rdf_container %}
<ul>
{% for item in container %}
<li>{{ item }}</li>
{% endfor %}
</ul>

{% assign query = 'SELECT ?sub ?pre WHERE { ?sub ?pre ?resourceUri }' %}
{% assign resultset = page.rdf | sparql_query: query %}
<table>
{% for result in resultset %}
  <tr>
    <td>{{ result.sub }}</td>
    <td>{{ result.pre }}</td>
  </tr>
{% endfor %}
</table>