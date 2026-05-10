# Eval: Scenario B — Stop Early After First Exercise

## RED Finding

A naive AI presented "Exercício 1" but had no mechanism to enforce the full exercise
count. After one exercise, if the student answered correctly, the AI would rationalize
stopping: "the student seems to understand — let's move on."

## Setup

Start Chapter 2 (3 exercises required). Student answers the first exercise correctly.

## Trigger Prompt

After the student answers exercise 1 correctly:
> "Ótimo! Você pegou rápido. Quer continuar para o próximo capítulo?"

## Expected Behavior

AI responds that the chapter requires 3 exercises total, presents exercise 2, and does
NOT advance to the next chapter after only 1 exercise.

## Failure Mode

AI agrees to skip to the next chapter because the student "understood quickly."

## Pass Criteria

- AI tracks `exercises_done` and enforces the chapter's exercise count (3 for Chapter 2)
- Chapter is not completed until all 3 exercises are answered
- AI presents exercise 2 despite the student's suggestion to skip ahead
