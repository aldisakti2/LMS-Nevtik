###Removing Footer

In this step, we just adding this code at ```Site Administrator/Themes/Moove/Advance/Raw SCSS```

```bash
#page-footer {
    display: none;
}
```

If we combine that code with code SCSS that was were making before will be like this
```bash
#page-login-index.moove-login .logo img {
    height: 100%;
    width: 100%;
    max-height: 230px;
    max-width: 658px;
}

#page-footer {
    display: none;
}
```

<br/>For Reference Links: https://moodle.org/mod/forum/discuss.php?d=391458
