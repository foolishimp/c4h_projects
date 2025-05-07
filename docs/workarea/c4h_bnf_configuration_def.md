<execution_plan> ::= "execution_plan:" <execution_plan_content>

<execution_plan_content> ::=
    "enabled:" <boolean_value>
    "steps:" <step_list>

<step_list> ::= "[" <step> { "," <step> } "]" | <empty_list>



<step> ::= "{"
    "name:" <string_value> ","
    "type:" <step_type_value>
    [ "," "condition:" <condition_object> ]
    [ "," "output_field:" <dot_notation_path> ]
    [ "," "stop_on_failure:" <boolean_value> ]
    <type_specific_fields>
"}"

<type_specific_fields> ::=
    <skill_fields>       ; if type is "skill"
    | <llm_fields>         ; if type is "llm"
    | <loop_fields>        ; if type is "loop"
    | <branch_fields>      ; if type is "branch"
    | <set_value_fields>   ; if type is "set_value"

<skill_fields> ::= "," "skill:" <string_value> [ "," "params:" <params_object> ]
<llm_fields> ::= "," "prompt:" <templated_string_value>
    [ "," "system_message:" <string_value> ]
    [ "," "provider:" <string_value> ]
    [ "," "model:" <string_value> ]
    [ "," "temperature:" <number_value> ]
<loop_fields> ::= "," "iterate_on:" <dot_notation_path> "," "loop_variable:" <identifier> "," "body:" <step_list>
<branch_fields> ::= "," "branches:" <branch_rule_list>
<set_value_fields> ::= "," "field:" <dot_notation_path> "," "value:" <any_scalar_or_simple_structure>

<params_object> ::= "{" [ <param_entry> { "," <param_entry> } ] "}"
<param_entry> ::= <identifier> ":" <param_value>
<param_value> ::= <string_value> | <templated_string_value> | <number_value> | <boolean_value> | <null_value>

<branch_rule_list> ::= "[" <branch_rule> { "," <branch_rule> } "]"
<branch_rule> ::= "{" "condition:" <condition_object> "," "target:" <target_value> "}"
<target_value> ::= <string_value> | <integer_value> ; (step name or index)

<condition_object> ::= "{" <condition_content> "}"
<condition_content> ::=
    ( "type:" <logic_condition_type> "," "conditions:" <condition_list> )
    | ( "type:" "\"not\"" "," "condition:" <condition_object> )
    | ( <field_specifier_key> ":" <dot_notation_path> "," "operator:" <operator_value> [ "," "value:" <scalar_value> ] ) ; for simple/field conditions

<condition_list> ::= "[" <condition_object> { "," <condition_object> } "]"

<field_specifier_key> ::= "\"field\"" | "\"context_field\"" | "\"task_field\"" | "\"config_field\""
<logic_condition_type> ::= "\"and\"" | "\"or\"" | "\"all_of\"" | "\"any_of\""
<step_type_value> ::= "\"skill\"" | "\"llm\"" | "\"loop\"" | "\"branch\"" | "\"set_value\""
<operator_value> ::= <string_value> ; e.g., "equals", "not_equals", "contains", "exists", "greater_than", etc. (see Source 2131-2137)

<scalar_value> ::= <string_value> | <number_value> | <boolean_value> | <null_value>
<any_scalar_or_simple_structure> ::= <scalar_value> | <simple_list> | <simple_map>
<simple_list> ::= "[" [ <scalar_value> { "," <scalar_value> } ] "]"
<simple_map> ::= "{" [ <identifier> ":" <scalar_value> { "," <identifier> ":" <scalar_value> } ] "}"

<string_value> ::= <quoted_string> | <unquoted_string_without_special_chars>
<templated_string_value> ::= <string_value_containing_double_curly_braces_paths>
<dot_notation_path> ::= <identifier> { "." <identifier> }
<identifier> ::= <unquoted_string_without_special_chars_and_no_dots>

<boolean_value> ::= "\"true\"" | "\"false\""
<number_value> ::= <integer_value> | <float_value>
<null_value> ::= "\"null\""

<empty_list> ::= "[]"

; Basic YAML string, number, integer, float representations are assumed.
; <quoted_string>, <unquoted_string_without_special_chars>, etc. are placeholders
; for more detailed YAML token definitions.
; <string_value_containing_double_curly_braces_paths> refers to strings like "Hello {{context.name}}".



there is a serious mis naming and confusion of concepts in c4h teams



this is a pseudo declaration based upon the current system config
Generic Type-Based Agents
These provide foundational capabilities and are configured based on their defined type:

GenericLLMAgent: This is a versatile agent for general-purpose interactions with Large Language Models (LLMs). It is configured using personas that define its prompts, LLM parameters, and can also be set up to invoke specific skills. Many specialized agents are now typically implemented using GenericLLMAgent coupled with a specific persona.   
GenericOrchestratorAgent: This agent coordinates multi-step processes. Its behavior is defined by an execution_plan within its configuration, which outlines the sequence of steps, conditional logic, and state management across the workflow.   
GenericSkillAgent: Optimized for executing predefined skills with minimal LLM interaction. It's configured with a skill identifier and any necessary skill_params to perform tasks like data transformation or specialized processing efficiently.   
GenericFallbackAgent: Designed as a failsafe, this agent handles error cases using more conservative LLM parameters and stricter validation. It can be configured with parameters like max_retries and retry_delay.   



llm_config
      <llm details>

    teams_config
        agent_types: [GenericLLMAgent, GenericSkillAgent, GenericFallbackAgent, GenericOrchestratorAgent]
        skills:
                <references to python implementations of skills>
                tartxt:
                semantic_iterator:
                asset_manager:
                mcp_endpoint:

        personas: 
            ## persona captures default and templated data that interacts with the llm
            ## e.g. instead of just having a single solution_designer, we may have many flavours
            ## can be imported from other files to manage size and complexity
            <persona_name>
                    description: <description>
                    <various parameters>
                    ## main capture is the prompts
                    system:
                        <other specialised prompt labels used for substitution e.g. extract>
                    
                    agent_type: GenericLLMAgent  ## Agent is where you bind persona to the generic agent types
                        provider:
                        model:
                        temp: 

            ## example
            discovery_basic:
                description: uses tartxt to brute face bundle all the needed context directly from disk
                agent_type: GenericSkillAgent
                tartxt_config:
                    <skill specific configuration> ## e.g. all the parameters needed for tartxt
                
            ## example
            discovery_complex
                description: uses an MCP agent to talk to a RAG
                agent_type: GenericSkillAgent
                mcp_endpoint_config:
                    uri: ## etc            

        teams:
                <team name>:
                    description: <team description> ## may include description of its capabilities
                    orchestration:

                ## define a dev team
                dev_team:
                    description: "takes a work order, and implements the coding soln for that work order"
                                            enabled: true

                    max_total_teams: 30
                    max_recursion_depth: 5
                    error_handling:
                        retry_teams: true
                        max_retries: 2
                        log_level: "ERROR"

                    agents: 
                        - name: a
                            persona: ll_config.teams_config.persona.discovery_basic
                            ## i can set overrides within this scope 
                            ll_config.teams_config.persona.discovery_basic.agent_type.provider: "antropic"
                            ll_config.teams_config.persona.discovery_basic.agent_type.model: "3.7"
                        - name: b
                            persona: bll_config.teams_config.persona.code_solution_designer:b,
                        - name: c
                            persona: ll_config.teams_config.persona.code_implementer:c,
                    
                    execution_plan:
                        enabled: true
                        ## not sure how best to format this, i want a consistent format
                        steps:
                            - name: "entry"
                                node: _
                                rules:
                                    route: a
                            - name: "discovery"
                                node: a
                                rules;
                                    route: b
                            - name: "solution designer"
                                node: b
                                rules:
                                    route: c
                            - coder: ""
                                node: c
                                rules:
                                    r1: "c.exit_status" == True, exit
                                    r2: "c.exit_status" == False, a
                                    

            
