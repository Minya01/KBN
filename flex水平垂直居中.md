##1
```
<div class="flexbox-container">
	<div>Blah blah</div>
	<div>Blah blah blah blah blah ...</div>
</div>
```
```
.flexbox-container {
	display: -ms-flexbox;
	display: -webkit-flex;
	display: flex;

	-ms-flex-align: center;
	-webkit-align-items: center;
	-webkit-box-align: center;

	align-items: center;
}
```

##2
```
<div class="Aligner">
  <div class="Aligner-item Aligner-item--top">…</div>
  <div class="Aligner-item">…</div>
  <div class="Aligner-item Aligner-item--bottom">…</div>
</div>
```
```
.Aligner {
  display: flex;
  align-items: center;
  justify-content: center;
}

.Aligner-item {
  max-width: 50%;
}

.Aligner-item--top {
  align-self: flex-start;
}

.Aligner-item--bottom {
  align-self: flex-end;
}
```
