# Discogs Scripts

Useful scripts than you can use on Discogs Website in navigator's console.

## [MyWantlist] Hide items not selling

[Example](https://www.discogs.com/mywantlist)

```js
document.querySelectorAll('tr.shortcut_navigable').forEach(el => {
    el.style.display = ['En Vente', 'For Sale'].some(x => el.innerText.includes(x)) ? '' : 'none'
})
```

## [Artist] Show only what you want

[Example](https://www.discogs.com/fr/artist/244819-In-Flames)

```js
(async (keys) => {
    document.querySelectorAll('.mr_toggler.button.button-white.button-short').forEach(x => x.click())
    await new Promise(resolve => setTimeout(resolve, 5000))
    document.querySelectorAll('tr.card.r_tr.sub:not(.hidden)').forEach(x => {
        x.classList.remove('sub_last')
        x.classList.remove('sub_first')
        x.style.display = keys.some(key => x.innerText.includes(key)) ? 'table-row' : 'none'
    })
})(['LP'])
```

## [Marketplace] Hide items "Various"

[Example](https://www.discogs.com/fr/sell/mywants)

Tips: You might need to hide "Errors" in console, these errors are issued by Discogs Website

```js
document.querySelectorAll('.shortcut_navigable').forEach(el => {
    el.style.display = el.querySelector('a.item_description_title').textContent.includes("Various") ? "none" : undefined
})
```

## [Marketplace] Hide items that are already in your collection

[Example](https://www.discogs.com/fr/sell/list?artist_id=244819)

Tips: You might need to hide "Errors" in console, these errors are issued by Discogs Website

```js
(async (usr) => {
    /** @type {string} Username */
    const username = document.querySelector('.user_image')?.alt ?? usr

    if (!username)
        throw 'You must log in'

    //Get number pages
    const pages = await fetch(
        `https://api.discogs.com/users/${username}/collection/folders/0/releases?per_page=25`, {
        method: 'GET',
        mode: 'cors',
        cache: 'default'
    })
        .then(response => response.json())
        .then(json => Math.ceil(json.pagination.items / 500))
        .catch(console.error)

    //Prepare requests
    const requests = []
    for (let i = 0; i < pages; i++) requests.push(
        fetch(`https://api.discogs.com/users/${username}/collection/folders/0/releases?per_page=500`, {
            method: 'GET',
            mode: 'cors',
            cache: 'default'
        })
        .catch(console.error)
    )

    //Get data
    await Promise.all(requests)
        .then(async (responses) => await Promise.all(responses.map(async (response) => await response.json())))
        .then(jsons => {
            ids = jsons.map(json => json.releases?.map(x => x.id?.toString()) ?? []).flat()
            ;[...document.querySelectorAll('.shortcut_navigable')].forEach(el => {
                el.style.display = ids.includes(el.querySelector('a.item_release_link').href.split('release/')?.[1]) ? "none" : undefined
            })
        })
        .catch(console.error)
})(undefined)
```
