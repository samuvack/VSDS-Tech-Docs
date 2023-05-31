
---
sort: 20
---

# RDF syntax


{% assign resource = '<http://www.ifi.uio.no/INF3580/simpsons#Homer>' | rdf_get %}
{{ resource | rdf_property: '<http://xmlns.com/foaf/0.1/job>' }}


<h1>{{ page.rdf | rdf_property: "rdfs:label", "en" }}</h1>
<div>{{ page.rdf | rdf_property: "dct:created" | date: "%Y-%m-%d" }}</div>
{% assign publicationlist = "ex:publicationlist" | rdf_container %}
<ul>
{% for pub in publicationlist %}
<li>{{ pub | rdf_property: "dc:title" }}</li>
<li>{{ pub | rdf_property: "dct:creator", false, true | join: ", " }}</li>
{% endfor %}
</ul>