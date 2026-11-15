<script lang="ts">
  import reports from '#lib/data/reports-4471.json';
  import notes from '#lib/data/desk-notes.json';
  import { groupByCause, readAs, uniqueById, type Report } from '#lib/kb/causes';
  import { tenantOf } from '#lib/kb/locales';

  let { ticket }: { ticket: number } = $props();

  const ticketRows = (reports as Report[]).filter((r) => r.mergedInto === ticket);
  const all = uniqueById(reports as Report[]).filter((r) => r.mergedInto === ticket);
  const groups = groupByCause(all, tenantOf);
  const rowCount = (channel: string, rows: Report[]) => rows.filter((r) => r.channel === channel).length;
  const tags = (counts: Record<string, number>) =>
    Object.entries(counts)
      .sort((a, b) => b[1] - a[1])
      .map(([tag, count]) => `${tag} ×${count}`)
      .join(' · ');
  const tenants = (rows: Report[]) => [...new Set(rows.map((r) => r.tenant))].sort().join(', ');
  const movedCount = groups.reduce((n, g) => n + readAs(g, notes.oneDayMove), 0);
</script>

<p class="summary">
  <strong>{ticketRows.length} export rows</strong> deduplicate to
  <strong>{all.length} unique reports</strong> in #{ticket} across
  <strong>{groups.length} causes</strong>.
</p>

<table>
  <thead>
    <tr><th>cause</th><th>reports</th><th>tenants</th><th>channels</th><th>intake tags (not cause)</th></tr>
  </thead>
  <tbody>
    {#each groups as g (g.cause)}
      <tr>
        <td><code>{g.cause}</code></td>
        <td>{g.reports.length}</td>
        <td>{g.tenants}: {tenants(g.reports)}</td>
        <td>app {rowCount('app', g.reports)} · email {rowCount('email', g.reports)} · form {rowCount('form', g.reports)}</td>
        <td>{tags(g.taggedAs)}</td>
      </tr>
    {/each}
  </tbody>
</table>

{#if movedCount}
  <p class="note">{movedCount} one-day-move reports remain in <code>parser</code> by the tenant-based cause rule; the symptom does not make a third cause.</p>
{/if}

<style>
  .summary { font-size: 1.1rem; }
  .note { color: var(--muted); font-size: 0.9rem; }
  td { vertical-align: top; }
</style>
