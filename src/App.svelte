<script>
    import DragDrop from "./demo.svelte";

    let data1 = ["Adams", "Boston", "Chicago", "Denver"];
		let data2 = [
				{"id": 0, "text": "Boston"}, 
				{"id": 1, "text": "Boston"}, 
				{"id": 2, "text": "Chicago"}, 
				{"id": 3, "text": "Denver"}];
		let data3 = [
				{"id": 0, "text": "Adams"},
				{"id": 1, "text": "Boston"}, 
				{"id": 2, "html": "<p style='color: red;'>Chicago</p>"}, 
				{"id": 3, "html": "<p style='color: blue;'>Denver</p>"}];

		let data = data1;
</script>

<style>
	main {
		text-align: center;
		display:flex;
	}

	cont {
		width:100px;
		height:100px;
	}
</style>

<main>
	<cont>
		<DragDrop bind:data={data}/>
	</cont>
</main>


