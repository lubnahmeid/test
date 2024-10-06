

# Documentation for `test_tweaks` Module

This document provides detailed documentation for the hooks implemented in the `test_tweaks` module, specifically for `test_tweaks_entity_presave()` and `test_tweaks_views_pre_build()` functions.

---

## 1. Function: `test_tweaks_entity_presave(EntityInterface $entity)`

### Purpose:
This hook implementation (`hook_entity_presave()`) is triggered before an entity is saved in Drupal. It specifically targets nodes of type **document**, **video**, and **classification_record** and performs certain actions to automatically update fields based on the node’s content.

### Parameters:
- **EntityInterface $entity**: The entity object that is being saved. The entity is typically a node, but this can also apply to other entity types.

### Description:
The function performs different actions based on the entity's bundle type:

### Use Case 1: Nodes of Type Document or Video
1. **Check the Bundle**:
   - If the entity's bundle is either **document** or **video**, the hook proceeds with this case.

2. **Extract the Year**:
   - The node’s created field (the creation timestamp) is used to extract the year.

3. **Check for Existing Taxonomy Term**:
   - The hook checks if a taxonomy term with the extracted year already exists in the **year** vocabulary.

4. **Create or Assign the Taxonomy Term**:
   - If the term doesn't exist, a new term for that year is created and saved in the **year** vocabulary.
   - If the term already exists, it retrieves the term ID.

5. **Set the Term in the Node**:
   - The taxonomy term (either created or existing) is then assigned to the node’s `field_year` field.

**Example**:
- A node of type **document** created on March 15, 2023, will have the year 2023 automatically assigned to the `field_year` field. If a term for 2023 doesn't exist in the **year** vocabulary, a new term is created.

### Use Case 2: Nodes of Type Classification Record
1. **Check the Bundle**:
   - If the entity's bundle is **classification_record**, the hook proceeds with this case.

2. **Retrieve Field Values**:
   - The hook loads the taxonomy term referenced in `field_classification_code` and retrieves its name.
   - Similarly, it loads the term referenced in `field_code_suffix (if present) and retrieves its name.

3. **Construct the Full Classification Code**:
   - If both fields are populated, it constructs a code in the format **classification_code(suffix)**. If no suffix is present, only the classification code is used.

4. **Set the Full Classification Code**:
   - The constructed code is then set in the `field_full_class_code` field of the node.

**Example**:
- If `field_classification_code` contains the term "A123" and `field_code_suffix` contains the term "XY," the full classification code will be set as "A123(XY)."

### Edge Cases:
- The hook first checks if fields like `field_classification_code` or `field_code_suffix` are populated (i.e., not empty). If any of these fields are empty, the hook skips the logic for that specific field.
- For document and video bundles, the hook handles the case where a taxonomy term for the year already exists, ensuring no duplicate terms are created.

### Key Functions Used:
- `\Drupal::entityTypeManager()->getStorage('taxonomy_term')->loadByProperties()`: Loads taxonomy terms based on specific properties (like name and vocabulary).
- `Term::create()`: Creates a new taxonomy term entity.
- `Term::load($term_id)`: Loads a taxonomy term based on its ID.
- `$entity->set()`: Sets a value for a field on the entity being saved.

### Dependencies:
- **Entity API**: This hook is dependent on the Drupal entity system, specifically for nodes.
- **Taxonomy Module**: The code assumes the use of Drupal's Taxonomy module for managing terms in the **year** vocabulary and fields like `field_classification_code`.

### Assumptions:
- `field_year`, `field_classification_code`, `field_code_suffix`, and `field_full_class_code` are existing fields on the relevant content types.
- The **year** vocabulary exists and is being used to categorize nodes based on the year of creation.
- The taxonomy terms referenced by `field_classification_code` and `field_code_suffix` exist and are valid.

---

## 2. Function: `test_tweaks_views_pre_build(ViewExecutable $view)`

### Purpose:
This function is an implementation of the `hook_views_pre_build()` in Drupal. It is used to dynamically modify the arguments of a specific view (**solr_search**) before the view query is built, based on the taxonomy term provided in the URL.

### Parameters:
- **ViewExecutable $view**: The view that is being built. This parameter contains all the data related to the view, including its ID, displays, and arguments.

### Description:
The function checks if the view being built is **solr_search** with the display **block_document_term_listing**. If so, it:

1. Retrieves the taxonomy term ID from the current page's URL.
2. Loads all child terms for the current taxonomy term.
3. Alters the view's contextual filter arguments:
   - If child terms exist, the arguments are modified to include both the child terms and the current term.
   - If no child terms exist, only the current term is included in the arguments.

This function helps refine search results in views that are based on taxonomy terms by ensuring that both parent and child terms are considered in the filtering process.

### Key Components:
1. **hook_views_pre_build()**:
   - This hook allows developers to modify a view just before the query is executed. It’s useful for altering how the query is built, adding or modifying filters, arguments, or conditions.

2. **$view->id() and $view->current_display**:
   - `id()`: gets the unique identifier of the view (**solr_search** in this case).
   - `current_display`: identifies the current display being rendered (e.g., **block_document_term_listing**).

3. **\Drupal::routeMatch()**:
   - This service retrieves information about the current route (i.e., the page's URL).
   - `getRawParameter('taxonomy_term')`: extracts the taxonomy term ID from the route parameters.

4. **Taxonomy Term Storage**:
   - `\Drupal::service('entity_type.manager')->getStorage('taxonomy_term')`: Loads the taxonomy term storage service, which is used to interact with taxonomy term entities.

5. **loadChildren()**:
   - Loads child terms of the current taxonomy term. This function returns an array of child term IDs, which are used to modify the view's arguments.

6. **Contextual Filter Arguments**:
   - `implode("+", $children) . "+" . $tid`: Joins the child term IDs with the current term ID to form a string. The `+` is used as a separator for contextual filters in the view.

### Use Cases:
This function is useful in scenarios where a view is filtering results based on taxonomy terms, and you want to ensure that child terms are considered in the filter. For instance:

1. **Search Views**: When filtering search results by categories, this ensures that selecting a parent category will also show results from its subcategories.
2. **Solr Search**: This is specifically targeting a Solr-based search view, where including child terms in the query can improve the relevance of search results.

### Related Documentation:
1. **hook_views_pre_build()**: Drupal API documentation
2. **Taxonomy Term Storage**: Entity Type Manager and Taxonomy Term Storage
3. **Route Matching**: `\Drupal::routeMatch()`

---
