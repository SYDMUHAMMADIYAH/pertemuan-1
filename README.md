# PEMBAGIAN TUGAS PROYEK

<!doctype html>
<html>
<head>
  <meta charset="utf-8" />
  <title>Preview Google Sheets</title>
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <style>
    body{font-family:system-ui,-apple-system,Segoe UI,Roboto,Helvetica,Arial,sans-serif;margin:20px}
    table{border-collapse:collapse;width:100%;max-width:1200px}
    th,td{border:1px solid #ddd;padding:8px;text-align:left;vertical-align:top}
    thead th{background:#f6f8fa;font-weight:600}
    tr:nth-child(even){background:#fafafa}
    caption{font-size:1.1rem;margin-bottom:8px}
    .error{color:#900}
  </style>
</head>
<body>
  <h2>Preview Sheet</h2>
  <div id="table">Memuat...</div>

  <script>
    const CSV_URL = "https://docs.google.com/spreadsheets/d/e/2PACX-1vS1THMl94BR2cyKvsVy9h8OElZlUHkD6u1qjh1E3roP6H0zYYT4dd9oTPY7lamcw39y4xDt0AiejlDk/pub?gid=89798501&single=true&output=csv";

    fetch(CSV_URL)
      .then(res => {
        if (!res.ok) throw new Error('Gagal memuat data: ' + res.status);
        return res.text();
      })
      .then(csv => {
        if (!csv.trim()) { document.getElementById('table').textContent = 'CSV kosong'; return; }

        // Split CSV into rows, handle quoted commas
        const rows = csv.trim().split('\n').map(r => r.match(/(".*?"|[^",\s]+)(?=\s*,|\s*$)/g)?.map(cell => {
          if (!cell) return '';
          // remove surrounding quotes if present, and unescape double quotes
          if (cell.startsWith('"') && cell.endsWith('"')) {
            return cell.slice(1,-1).replace(/""/g, '"');
          }
          return cell;
        }) || []);

        const header = rows.shift() || [];
        let html = ['<table>','<caption>Data terbaru dari Google Sheets</caption>','<thead><tr>'];
        header.forEach(h => html.push('<th>' + escapeHtml(h) + '</th>'));
        html.push('</tr></thead><tbody>');
        rows.forEach(r => {
          html.push('<tr>');
          header.forEach((_, i) => html.push('<td>' + escapeHtml(r[i] ?? '') + '</td>'));
          html.push('</tr>');
        });
        html.push('</tbody></table>');
        document.getElementById('table').innerHTML = html.join('');
      })
      .catch(err => {
        document.getElementById('table').innerHTML = '<div class="error">Error: ' + escapeHtml(err.message) + '</div>';
      });

    function escapeHtml(s){ return String(s).replace(/&/g,'&amp;').replace(/</g,'&lt;').replace(/>/g,'&gt;'); }
  </script>
</body>
</html>
