# Dumping a tab for editing

Run this after `Google Docs:read_doc` saved its result to a file. It prints the
revision id, the tabs in order, and every non-empty paragraph of the first tab
(the newest) with its index range, paragraph style and bullet list id. Pass the
saved file as the only argument; add search terms after it to print, for the
matching paragraphs only, each text run with its own indices and inline styles
(bold, link, highlight), which is what `replaceAllText` and `updateTextStyle`
need.

```bash
python3 -I - "$FILE" "term one" "term two" <<'PY'
import json, sys
path, *terms = sys.argv[1:]
doc = json.load(open(path))['content']
print('revision', doc['revisionId'])
for tab in doc['tabs']:
    props = tab['tabProperties']
    print(props['tabId'], props['index'], props['title'])

def text(paragraph):
    return ''.join(e['textRun']['content'] for e in paragraph['elements'] if 'textRun' in e)

newest = doc['tabs'][0]
print('\n=== newest tab', newest['tabProperties']['tabId'])
for block in newest['documentTab']['body']['content']:
    if 'paragraph' not in block:
        continue
    paragraph = block['paragraph']
    content = text(paragraph).rstrip('\n')
    if not content.strip():
        continue
    style = paragraph['paragraphStyle'].get('namedStyleType')
    bullet = (paragraph.get('bullet') or {}).get('listId')
    print(f"[{block['startIndex']}-{block['endIndex']}] {style} bullet={bullet} {content}")
    if terms and any(t in content for t in terms):
        for e in paragraph['elements']:
            if 'textRun' not in e:
                continue
            ts = e['textRun']['textStyle']
            flags = {k: v for k, v in ts.items() if k in ('bold', 'link', 'backgroundColor')}
            print(f"    run [{e['startIndex']}-{e['endIndex']}] {flags} {e['textRun']['content']!r}")
PY
```

`-I` isolates Python from the current directory, since the saved file is
downloaded data. The script only reads; nothing it prints is written back.
