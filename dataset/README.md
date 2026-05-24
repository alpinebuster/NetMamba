## Data Preprocessing
Split individual pcap files
For `x` in [`cic_iot2022`, `cross_platform`, `iscx_tor2016`, `iscx_vpn2016`, `ustc_tfc2016`], download corresponding pcap files from their official websites, and then run the following command to execute `Flow Splitting`

```sh
python dataset_x.py
```

To porcess all datasets, run `python dataset_all.py --all`
- Sample with lower bound and upper bound
    ```sh
    python dataset_all.py --sample
    ```
- Execute `Packet Parsing`, `Packet Cropping & Padding`, and `Packet Concatenating`
    ```sh
    python dataset_all.py --array
    ```
- Split train, valid, and test sets for fine-tuning
    ```sh
    python dataset_all.py --split
    ```
- Merge all datasets for pre-training
    ```sh
    python dataset_all.py --merge
    ```

## Dependency
- `editcap`: For changing `.pcapng` into `.pcap` formats, see [editcap](https://www.wireshark.org/docs/man-pages/editcap.html).
- `mergecap`: For merging individual pcap files into an integrated one, see [mergecap](https://www.wireshark.org/docs/man-pages/mergecap.html).
- `splitter`: For efficiently splitting traffic flows from raw pcap files, this is a customized tool, see [ShieldGPT/pcap_tool](https://github.com/wangtz19/ShieldGPT/tree/master/pcap_tool)

### Run splitter
Run `./splitter -h` to get the following help information:
```text
Allowed options:
  -h [ --help ]            produce help message
  -i [ --input ] arg       set input file name
  -o [ --output_dir ] arg  set result directory
  -f [ --flow_format ] arg set format for splitting flows, available options: ip_pair, ip_pair_reverse, five_tuple
```

*flow format* explanation:
* `ip_pair`: the input pcap file is splitted by src & dst ip, and stored by `<src_ip>_<dst_ip>.pcap` format
* `ip_pair_reverse`: the input pcap file is splitted by src & dst ip, and stored by `<dst_ip>_<src_ip>.pcap` format
* `five_tuple`: the input pcap file is splitted by five tuple (ip, port & protocol), and stored by `<src_ip>_<src_port>_<proto>_<dst_ip>_<dst_port>.pcap` format
