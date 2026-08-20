# Pseudocode - EShop API Test Generator

```text
function generate_api_test_cases(spec_file, readme_file, assignment_file, target_endpoint):
    spec_text = read_file(spec_file)
    readme_text = read_file(readme_file)
    assignment_text = read_file(assignment_file)

    endpoint = parse_endpoint(spec_text, target_endpoint)
    if endpoint is not found:
        raise error("Target endpoint is not defined in specification")

    rules = extract_rules(endpoint)
    unknowns = identify_unknown_behavior(rules)

    partitions = build_parameter_partitions(rules.parameters)
    security_dimensions = build_security_dimensions(rules.auth, rules.authorization, rules.fields)
    schema_dimensions = build_schema_dimensions(rules.responses)
    state_dimensions = build_state_dimensions(rules.business_rules, rules.state_rules)

    coverage_plan = merge_dimensions(
        partitions,
        security_dimensions,
        schema_dimensions,
        state_dimensions,
        unknowns
    )

    candidate_cases = []
    for coverage_item in coverage_plan:
        case = create_case_from_coverage_item(coverage_item, endpoint, rules)

        case.case_id = allocate_case_id(target_endpoint)
        case.api = target_endpoint
        case.source = "AI_GENERATED"
        case.headers = include_required_headers(case.headers, "X-Student-Id: 23127364")
        case.audit_status = "PENDING_HUMAN_REVIEW"

        if expected_status_is_defined_by_spec(coverage_item, rules):
            case.expected_status = get_defined_status(coverage_item, rules)
        else:
            case.expected_status = "SPEC_UNDEFINED"
            case.assumption_or_open_question = explain_missing_spec_rule(coverage_item)

        case.spec_reference = find_strongest_spec_reference(coverage_item, endpoint)
        case.requirement_ref = find_requirement_reference(coverage_item, assignment_text)
        case.security_ref = find_security_reference_if_applicable(coverage_item)

        candidate_cases.append(case)

    candidate_cases = remove_duplicate_or_redundant_cases(candidate_cases)
    candidate_cases = prioritize_meaningful_breadth(candidate_cases)

    if count(candidate_cases) < 35:
        candidate_cases = add_more_valid_exploratory_cases(candidate_cases, rules, unknowns)

    validate_output(candidate_cases, target_endpoint)

    return export_spreadsheet_ready_rows(candidate_cases)
```

## Hàm kiểm tra output

```text
function validate_output(candidate_cases, target_endpoint):
    assert all case_id values are unique
    assert every case.api equals target_endpoint
    assert every case.source equals "AI_GENERATED"
    assert every case.audit_status equals "PENDING_HUMAN_REVIEW"
    assert no case claims executed, passed, failed, or bug_found
    assert every case has objective, request method, request URL, headers, and assertion
    assert every executable request includes X-Student-Id

    for each case in candidate_cases:
        if case uses unsupported field:
            assert case.coverage_type contains "security" or "exploratory"
        if case.expected_status is concrete:
            assert status is supported by specification or unambiguous requirement
        if case.expected_status equals "SPEC_UNDEFINED":
            assert assumption_or_open_question is not empty
```

## Luồng human audit sau generation

```text
function process_human_audit(audited_workbook):
    rows = read_workbook(audited_workbook)

    for each row in rows:
        preserve_original_ai_content(row)
        preserve_human_audit_status(row)
        preserve_human_audit_notes(row)

        if row.audit_status is "INVALID" or "INCOMPLETE":
            require human_correction approved by student
            write correction to human_correction only

    do not change source
    do not redesign AI-generated cases
    do not execute tests
```

## Luồng manual extension

```text
function append_manual_extensions(audited_workbook, selected_student_gaps):
    rows = read_workbook(audited_workbook)

    for each selected_gap in selected_student_gaps:
        case = create_manual_extension_case(selected_gap)
        case.case_id = allocate_extension_id()
        case.source = "MANUAL_EXTENSION"
        case.audit_status = "STUDENT_DESIGNED"
        case.traceability = link_to_gap_analysis_and_spec()
        case.headers include X-Student-Id

        append case to rows

    save workbook
    do not execute tests
```
