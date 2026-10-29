<script lang="ts">
  import reports from '#lib/data/reports-4471.json';
  import notes from '#lib/data/desk-notes.json';
  import { groupByCause, readAs, uniqueById, type Report } from '#lib/kb/causes';
  import { tenantOf } from '#lib/kb/locales';

  // One row per problem: the desk's causes as CauseTable counts them, plus the reports the desk
  // read as a one-day move. Those stay counted where they're filed. `issues` maps a problem to its ticket link.
  let { ticket, issues = {} }: { ticket: number; issues?: Record<string, string> } = $props();

  const merged = $derived(uniqueById(reports as Report[]).filter((r) => r.mergedInto === ticket));
  const groups = $derived(groupByCause(merged, tenantOf));
  const oneDay = $derived(merged.filter((r) => notes.oneDayMove.includes(r.id)));
  const filedUnder = $derived(groups.find((g) => readAs(g, notes.oneDayMove))?.cause);
  const rows = $derived([
    ...groups.map((g) => ({ problem: g.cause as string, reports: g.reports, note: '' })),
    { problem: 'one-day move', reports: oneDay, note: `also counted under ${filedUnder}` },
  ]);
  const tenants = (rs: Report[]) => new Set(rs.map((r) => r.tenant)).size;
  const seen = (rs: Report[]) => {
    const days = rs.map((r) => r.receivedAt.slice(0, 10)).sort();
    return `${days[0]} → ${days[days.length - 1]}`;
  };
</script>

<p class="sum">
  <strong>{merged.length}</strong> reports merged into #{ticket} ·
  <strong>{rows.length}</strong> problems
</p>

<table>
  <thead>
    <tr><th>problem</th><th>reports</th><th>tenants</th><th>seen</th><th>ticket</th></tr>
  </thead>
  <tbody>
    {#each rows as row (row.problem)}
      <tr>
        <td><code>{row.problem}</code></td>
        <td>{row.reports.length}{#if row.note}<span class="note"> · {row.note}</span>{/if}</td>
        <td>{tenants(row.reports)}</td>
        <td>{seen(row.reports)}</td>
        <td>{#if issues[row.problem]}<a href={issues[row.problem]}>#{issues[row.problem].split('/').pop()}</a>{:else}—{/if}</td>
      </tr>
    {/each}
  </tbody>
</table>

<style>
  .sum { font-size: 1.1rem; }
  .note { color: var(--muted); font-size: 0.85em; }
</style>
