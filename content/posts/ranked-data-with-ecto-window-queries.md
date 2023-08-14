+++
title = "Ranked data with Ecto window queries"
date = "2022-03-20T13:00:46+04:00"
author = "Danny Hawkins"
authorTwitter = "dannyhawkins"
categories = ["coding"]
tags = ["elixir", "ecto", "data", "postgres"]
showFullContent = false
readingTime = true
hideComments = false
+++

I recently came across a problem that I would have usually thought to solve by adding additional field(s) to the database to track the rank of a row of data within its table. However having learned about [window functions](https://hexdocs.pm/ecto/Ecto.Query.WindowAPI.html) recently, I realised I could solve this in a much more elegant way using window functions and [subqueries](https://hexdocs.pm/ecto/Ecto.SubQuery.html) in Ecto

## The Problem

I have a set of data where there is a scoring element and a grouping element, for example:

```csv
| Name       | Region | Score |
| ---------- | ------ | ----- |
| AE Group 1 | UAE    | 10    |
| AE Group 2 | UAE    | 30    |
| AE Group 3 | UAE    | 35    |
| AE Group 4 | UAE    | 50    |
| UK Group 1 | UK     | 8     |
| UK Group 2 | UK     | 30    |
| UK Group 3 | UK     | 38    |
| UK Group 4 | UK     | 52    |
```

When presenting this data, I also want to be able to show a rank element, and depending on the filter I want to be able to show that position within the scope, for example when looking at all results, the rank should be within the scope of all records:

```csv
| Rank | Name       | Region | Score |
| ---- | ---------- | ------ | ----- |
| 1    | UK Group 4 | UK     | 52    |
| 2    | AE Group 4 | UAE    | 50    |
| 3    | UK Group 3 | UK     | 38    |
| 4    | AE Group 3 | UAE    | 35    |
| 5    | AE Group 2 | UAE    | 30    |
| 6    | UK Group 2 | UK     | 30    |
| 7    | AE Group 1 | UAE    | 10    |
| 8    | UK Group 1 | UK     | 8     |
```

When filtering for **UK** the rank should be within the scope of the UK:

```csv
| Rank | Name       | Region | Score |
| ---- | ---------- | ------ | ----- |
| 1    | UK Group 4 | UK     | 52    |
| 2    | UK Group 3 | UK     | 38    |
| 3    | UK Group 2 | UK     | 30    |
| 4    | UK Group 1 | UK     | 8     |
```

When filtering for **UAE** the rank should be in the scope of UAE:

```csv
| Rank | Name       | Region | Score |
| ---- | ---------- | ------ | ----- |
| 1    | AE Group 4 | UAE    | 50    |
| 2    | AE Group 3 | UAE    | 35    |
| 3    | AE Group 2 | UAE    | 30    |
| 4    | AE Group 1 | UAE    | 10    |
```

Finally to make things a little more interesting we will allow for a search to be made against the dataset, which will filter data at the database level (using a **WHERE** condition) but **not lose the ranking within the regional or global scope**. This is the part that made this particular solution effective.

```text
Filter: "UAE", Search: "Group 4"
```

```csv
| Rank | Name       | Region | Score |
| ---- | ---------- | ------ | ----- |
| 1    | AE Group 4 | UAE    | 50    |
```

```text
Filter: "UK", Search: "UK Group 2"
```

```csv
| Rank | Name       | Region | Score |
| ---- | ---------- | ------ | ----- |
| 2    | UK Group 3 | UK     | 38    |
```

```text
Filter: "All", Search: "Group 3"
```

```csv
| Rank | Name       | Region | Score |
| ---- | ---------- | ------ | ----- |
| 3    | UK Group 3 | UK     | 38    |
| 4    | AE Group 3 | UAE    | 35    |
```

## Solution

I’ll to go through the full solution here, but if you want to just head to the code, it’s on github [here](https://github.com/quiqupltd/ecto_window_functions_example) (follow the Getting started section in the README.)

First of all lets create our migration and schema, we’re creating and model and table for groups, that have a name, region and score.

```elixir
# priv/repo/migrations/20220319222511_create_groups.exs

defmodule TestApp.Repo.Migrations.CreateGroups do
  use Ecto.Migration

  def change do
    create table(:groups) do
      add :name, :string, null: false
      add :region, :string, null: false
      add :score, :integer, null: false, default: 0
    end

    create index(:groups, :region)
  end
end
```

```elixir
# lib/test_app/groups/group.ex

defmodule TestApp.Groups.Group do
  use Ecto.Schema
  import Ecto.Changeset

  schema "groups" do
    field(:name, :string, null: false)
    field(:region, :string, null: false)
    field(:score, :integer, null: false)

    # Has to be virtual!
    field(:group_rank, :integer, virtual: true)
  end

  def changeset(group, attrs) do
    group
    |> cast(attrs, [:name, :region, :score])
    |> validate_required([:name, :region, :score])
  end
end
```

The main thing to note here is the virtual attribute **group_rank**, we’re going to use this later to hold a computed value for the rank

Next lets create a test suite that matches our brief:

```elixir
# test/support/fixtures/group_fixtures.exs

defmodule TestApp.GroupFixtures do
  @doc """
  Generate groups.
  """
  def group_fixtures() do
    [
      {"AE Group 1", "UAE", 10},
      {"AE Group 2", "UAE", 30},
      {"AE Group 3", "UAE", 35},
      {"AE Group 4", "UAE", 50},
      {"UK Group 1", "UK", 8},
      {"UK Group 2", "UK", 30},
      {"UK Group 3", "UK", 38},
      {"UK Group 4", "UK", 52}
    ]
    |> Enum.each(fn {name, region, score} ->
      TestApp.Groups.create_group(%{name: name, region: region, score: score})
    end)
  end
end
```

```elixir
# test/groups_test.exs

defmodule TestApp.GroupsTest do
  use TestApp.DataCase, async: true

  alias TestApp.Groups

  import TestApp.GroupFixtures

  setup do
    group_fixtures()
  end

  defp assert_ranked(results, expected) do
    assert Enum.map(results, fn group ->
             {group.group_rank, group.name, group.region, group.score}
           end) ==
             expected
  end

  describe "list_groups/2" do
    test "return the full list ranked" do
      assert_ranked(
        Groups.list_groups(),
        [
          {1, "UK Group 4", "UK", 52},
          {2, "AE Group 4", "UAE", 50},
          {3, "UK Group 3", "UK", 38},
          {4, "AE Group 3", "UAE", 35},
          {5, "AE Group 2", "UAE", 30},
          {6, "UK Group 2", "UK", 30},
          {7, "AE Group 1", "UAE", 10},
          {8, "UK Group 1", "UK", 8}
        ]
      )
    end

    test "return UAE region ranked" do
      assert_ranked(
        Groups.list_groups("UAE"),
        [
          {1, "AE Group 4", "UAE", 50},
          {2, "AE Group 3", "UAE", 35},
          {3, "AE Group 2", "UAE", 30},
          {4, "AE Group 1", "UAE", 10}
        ]
      )
    end

    test "return UK region ranked" do
      assert_ranked(
        Groups.list_groups("UK"),
        [
          {1, "UK Group 4", "UK", 52},
          {2, "UK Group 3", "UK", 38},
          {3, "UK Group 2", "UK", 30},
          {4, "UK Group 1", "UK", 8}
        ]
      )
    end

    test "return search result with correct rank for UAE" do
      assert_ranked(
        Groups.list_groups("UAE", "Group 4"),
        [
          {1, "AE Group 4", "UAE", 50}
        ]
      )
    end

    test "return search result with correct rank for UK" do
      assert_ranked(
        Groups.list_groups("UK", "Group 3"),
        [
          {2, "UK Group 3", "UK", 38}
        ]
      )
    end

    test "return search result with correct rank for full list" do
      assert_ranked(
        Groups.list_groups(nil, "Group 3"),
        [
          {3, "UK Group 3", "UK", 38},
          {4, "AE Group 3", "UAE", 35}
        ]
      )
    end
  end
end
```

Then create the code to make the tests pass, we want to break down the task into 3 main parts.

## Filter the Region

```elixir
def list_groups(region \\ nil, search \\ nil) do
  query =
    from(g in Group)
    |> filter_region(region)

  query |> Repo.all()
end

defp filter_region(query, nil), do: query

defp filter_region(query, region) do
  from(g in query, where: g.region == ^region)
end
```

## Rank the data using a window function

This is where we are using the virtual attribute from earlier **group_rank**, we populate it in the select using the window function [row_number](https://hexdocs.pm/ecto/Ecto.Query.WindowAPI.html#row_number/0)

```elixir
def list_groups(region \\ nil, search \\ nil) do
  query =
    from(g in Group)
    |> filter_region(region)
    |> add_rank(region)

  query |> Repo.all()
end

# When no region is past we don't need to partition, we have global rank
defp add_rank(query, nil) do
  from(g in query,
    windows: [p: [partition_by: nil, order_by: [desc: g.score]]],
    select_merge: %{group_rank: row_number() |> over(:p)}
  )
end

# When a region is passed we should partition by the region to establish a rank in each region
defp add_rank(query, _region) do
  from(g in query,
    windows: [p: [partition_by: g.region, order_by: [desc: g.score]]],
    select_merge: %{group_rank: row_number() |> over(:p)}
  )
end
```

## Search the data

Finally we want to run a search across the results if a search string is passed, the important part here is to put the query we have built so far into a subquery, so the the search is executed over the already ranked data, this way we don’t calculate the rank on the result after the search

```elixir
def list_groups(region \\ nil, search \\ nil) do
  query =
    from(g in Group)
    |> filter_region(region)
    |> add_rank(region)
    |> search(search)

  query |> Repo.all()
end

defp search(query, nil), do: query

# Make sure to use a subquery to we don't lose the correct ordering
defp search(query, search) do
  from(g in subquery(query), where: ilike(g.name, ^"%#{search}%"))
end
```

The final groups module looks looks like:

```elixir
# lib/groups.ex
defmodule TestApp.Groups do
  import Ecto.Query

  alias TestApp.Repo
  alias TestApp.Groups.Group

  @doc """
  Create a new group

  ## Examples

    iex> TestApp.Groups.create_group(%{name: "group1", region: "UAE", score: 10})
    {:ok, %TestApp.Groups.Group{name: "group1", region: "UAE", score: 10}
  """
  def create_group(attrs) do
    %Group{} |> Group.changeset(attrs) |> Repo.insert()
  end

  def list_groups(region \\ nil, search \\ nil) do
    query =
      from(g in Group)
      |> filter_region(region)
      |> add_rank(region)
      |> search(search)

    query |> Repo.all()
  end

  defp filter_region(query, nil), do: query

  defp filter_region(query, region) do
    from(g in query, where: g.region == ^region)
  end

  # When no region is past we don't need to partition, we have global rank
  defp add_rank(query, nil) do
    from(g in query,
      windows: [p: [partition_by: nil, order_by: [desc: g.score]]],
      select_merge: %{group_rank: row_number() |> over(:p)}
    )
  end

  # When a region is passed we should partition by the region to establish a rank in each region
  defp add_rank(query, _region) do
    from(g in query,
      windows: [p: [partition_by: g.region, order_by: [desc: g.score]]],
      select_merge: %{group_rank: row_number() |> over(:p)}
    )
  end

  defp search(query, nil), do: query

  # Make sure to use a subquery to we don't lose the correct ordering
  defp search(query, search) do
    from(g in subquery(query), where: ilike(g.name, ^"%#{search}%"))
  end
end
```

## Summary
Window functions and subquery options provide a great deal of power to work with data before it’s gets fetched directly into your runtime.

Typically developers who are less SQL savvy may solve this problem by adding additional columns to the database (efficient for queries), or pulling larger amounts of data from the database, the manipulating the results with code (over fetching data). I found this approach to be very pleasing and I’m already thinking of other areas that can be improved in code that I work on with better use of Ecto and SQL.
